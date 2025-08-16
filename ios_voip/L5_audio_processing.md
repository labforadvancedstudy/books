# L5: 실시간 음성 처리

## Core Insight
음성은 단순한 소리가 아니다. 감정, 의미, 정체성을 담은 복잡한 신호다. 실시간으로 이를 처리하는 것은 계산과 지연 사이의 끊임없는 줄타기다.

---

## 음성의 해부학

사람의 목소리는 어떻게 만들어질까?

```
성대 진동 (기본 주파수) → 성도 공명 (포먼트) → 입술/혀 조음 → 목소리
```

이를 디지털로:

```swift
struct VoiceCharacteristics {
    let fundamentalFrequency: Float  // 100-300 Hz (성대 진동)
    let formants: [Float]            // F1, F2, F3... (모음 특성)
    let harmonics: [Float]           // 배음 (음색)
    let noiseRatio: Float           // 숨소리, 치찰음
}

// 남성 평균: 125 Hz
// 여성 평균: 210 Hz  
// 아동 평균: 300 Hz
```

## 코덱: 압축의 예술

### Opus: 현대 VoIP의 표준

```swift
import OpusCodec

class OpusProcessor {
    private var encoder: OpaquePointer?
    private var decoder: OpaquePointer?
    
    init(sampleRate: Int32 = 48000, channels: Int32 = 1) {
        // 인코더 생성
        var error: Int32 = 0
        encoder = opus_encoder_create(sampleRate, channels, 
                                     OPUS_APPLICATION_VOIP, &error)
        
        // VoIP 최적화 설정
        opus_encoder_ctl(encoder, OPUS_SET_VBR(1))           // Variable Bitrate
        opus_encoder_ctl(encoder, OPUS_SET_VBR_CONSTRAINT(0)) // Unconstrained VBR
        opus_encoder_ctl(encoder, OPUS_SET_COMPLEXITY(5))     // 중간 복잡도
        opus_encoder_ctl(encoder, OPUS_SET_PACKET_LOSS_PERC(10)) // 10% 손실 예상
        opus_encoder_ctl(encoder, OPUS_SET_DTX(1))           // 무음 감지
        
        // 디코더 생성
        decoder = opus_decoder_create(sampleRate, channels, &error)
    }
    
    func encode(_ pcmData: [Int16]) -> Data? {
        let frameSize = 960  // 20ms at 48kHz
        var encoded = [UInt8](repeating: 0, count: 4000)
        
        let encodedBytes = opus_encode(
            encoder,
            pcmData,
            Int32(frameSize),
            &encoded,
            Int32(encoded.count)
        )
        
        if encodedBytes > 0 {
            return Data(encoded.prefix(Int(encodedBytes)))
        }
        return nil
    }
    
    func decode(_ opusData: Data) -> [Int16]? {
        var pcm = [Int16](repeating: 0, count: 5760)  // 120ms 버퍼
        
        let decodedSamples = opus_decode(
            decoder,
            opusData.withUnsafeBytes { $0.bindMemory(to: UInt8.self).baseAddress },
            Int32(opusData.count),
            &pcm,
            Int32(pcm.count / 2),  // stereo면 /2
            0  // FEC 사용 안함
        )
        
        if decodedSamples > 0 {
            return Array(pcm.prefix(Int(decodedSamples)))
        }
        return nil
    }
}
```

### 코덱 비교

```swift
enum AudioCodec {
    case opus       // 6-510 kbps, 최고 품질, 낮은 지연
    case g711       // 64 kbps, 전통 전화, 호환성 최고
    case g729       // 8 kbps, 저대역폭, 특허 문제
    case aac        // 가변, Apple 생태계
    case amrWB      // 6.6-23.85 kbps, 모바일 최적화
    
    var bitrate: ClosedRange<Int> {
        switch self {
        case .opus: return 6000...510000
        case .g711: return 64000...64000
        case .g729: return 8000...8000
        case .aac: return 16000...320000
        case .amrWB: return 6600...23850
        }
    }
    
    var latency: Int {  // milliseconds
        switch self {
        case .opus: return 5
        case .g711: return 0
        case .g729: return 15
        case .aac: return 100
        case .amrWB: return 25
        }
    }
}
```

## 에코 제거: AEC의 마법

에코는 VoIP의 숙적이다:

```swift
import Accelerate

class AcousticEchoCanceller {
    private let filterLength = 512
    private var adaptiveFilter: [Float]
    private var stepSize: Float = 0.01
    
    init() {
        adaptiveFilter = [Float](repeating: 0, count: filterLength)
    }
    
    // NLMS (Normalized Least Mean Squares) 알고리즘
    func cancelEcho(micSignal: [Float], 
                   speakerSignal: [Float]) -> [Float] {
        var output = [Float](repeating: 0, count: micSignal.count)
        
        for i in filterLength..<micSignal.count {
            // 1. 에코 추정
            let referenceSegment = Array(speakerSignal[(i-filterLength)..<i])
            var echoEstimate: Float = 0
            vDSP_dotpr(referenceSegment, 1, 
                      adaptiveFilter, 1, 
                      &echoEstimate, 
                      vDSP_Length(filterLength))
            
            // 2. 에코 제거
            let error = micSignal[i] - echoEstimate
            output[i] = error
            
            // 3. 필터 적응 (NLMS)
            var power: Float = 0
            vDSP_sve_sq(referenceSegment, 1, &power, vDSP_Length(filterLength))
            
            if power > 0.001 {  // 0으로 나누기 방지
                let normalizedStepSize = stepSize / (power + 0.001)
                
                // 필터 계수 업데이트
                var scaledError = error * normalizedStepSize
                vDSP_vsma(referenceSegment, 1, 
                         &scaledError, 
                         adaptiveFilter, 1, 
                         &adaptiveFilter, 1, 
                         vDSP_Length(filterLength))
            }
        }
        
        return output
    }
    
    // 더블 토크 감지 (양쪽이 동시에 말할 때)
    func detectDoubleTalk(nearEnd: [Float], 
                         farEnd: [Float]) -> Bool {
        var nearPower: Float = 0
        var farPower: Float = 0
        
        vDSP_sve_sq(nearEnd, 1, &nearPower, vDSP_Length(nearEnd.count))
        vDSP_sve_sq(farEnd, 1, &farPower, vDSP_Length(farEnd.count))
        
        let ratio = nearPower / (farPower + 0.001)
        
        // 비율이 특정 범위면 더블 토크
        return ratio > 0.3 && ratio < 3.0
    }
}
```

## 노이즈 제거: 깨끗한 음성

### Spectral Subtraction

```swift
class NoiseReducer {
    private let fftSize = 512
    private var noiseProfile: [Float]?
    private let fft = FFT(size: 512)
    
    // 노이즈 프로파일 학습 (무음 구간에서)
    func learnNoiseProfile(from silentAudio: [Float]) {
        let spectrum = fft.forward(silentAudio)
        noiseProfile = spectrum.magnitude
    }
    
    // 노이즈 제거
    func reduceNoise(in audio: [Float]) -> [Float] {
        guard let noise = noiseProfile else { return audio }
        
        // 1. 주파수 영역으로 변환
        let spectrum = fft.forward(audio)
        var magnitude = spectrum.magnitude
        let phase = spectrum.phase
        
        // 2. 스펙트럴 차감
        for i in 0..<magnitude.count {
            magnitude[i] = max(0, magnitude[i] - noise[i] * 1.5)
        }
        
        // 3. 다시 시간 영역으로
        return fft.inverse(magnitude: magnitude, phase: phase)
    }
}

// FFT 래퍼
class FFT {
    private let setup: FFTSetup
    private let log2n: vDSP_Length
    private let n: Int
    private let nOver2: Int
    
    init(size: Int) {
        self.n = size
        self.nOver2 = size / 2
        self.log2n = vDSP_Length(log2(Float(size)))
        self.setup = vDSP_create_fftsetup(log2n, FFTRadix(kFFTRadix2))!
    }
    
    func forward(_ input: [Float]) -> (magnitude: [Float], phase: [Float]) {
        var real = [Float](repeating: 0, count: nOver2)
        var imag = [Float](repeating: 0, count: nOver2)
        var splitComplex = DSPSplitComplex(realp: &real, imagp: &imag)
        
        input.withUnsafeBufferPointer { ptr in
            ptr.baseAddress!.withMemoryRebound(to: DSPComplex.self, 
                                              capacity: nOver2) { complexPtr in
                vDSP_ctoz(complexPtr, 2, &splitComplex, 1, vDSP_Length(nOver2))
            }
        }
        
        vDSP_fft_zrip(setup, &splitComplex, 1, log2n, FFTDirection(FFT_FORWARD))
        
        var magnitude = [Float](repeating: 0, count: nOver2)
        var phase = [Float](repeating: 0, count: nOver2)
        
        vDSP_zvabs(&splitComplex, 1, &magnitude, 1, vDSP_Length(nOver2))
        vDSP_zvphas(&splitComplex, 1, &phase, 1, vDSP_Length(nOver2))
        
        return (magnitude, phase)
    }
}
```

## VAD (Voice Activity Detection)

무음을 감지하여 대역폭 절약:

```swift
class VoiceActivityDetector {
    private let energyThreshold: Float = 0.01
    private let zeroCrossingThreshold = 20
    private var hangoverCounter = 0
    private let hangoverFrames = 5
    
    func detectVoice(in frame: [Float]) -> Bool {
        // 1. 에너지 기반 감지
        var energy: Float = 0
        vDSP_sve_sq(frame, 1, &energy, vDSP_Length(frame.count))
        energy = energy / Float(frame.count)
        
        // 2. Zero Crossing Rate
        var zcr = 0
        for i in 1..<frame.count {
            if (frame[i] >= 0) != (frame[i-1] >= 0) {
                zcr += 1
            }
        }
        
        // 3. 판단 로직
        let hasVoice = energy > energyThreshold && 
                      zcr > zeroCrossingThreshold
        
        // 4. Hangover (끝말 잘림 방지)
        if hasVoice {
            hangoverCounter = hangoverFrames
            return true
        } else if hangoverCounter > 0 {
            hangoverCounter -= 1
            return true
        }
        
        return false
    }
}
```

## AGC (Automatic Gain Control)

음량을 자동으로 조절:

```swift
class AutomaticGainControl {
    private let targetLevel: Float = 0.3
    private var currentGain: Float = 1.0
    private let attackTime: Float = 0.002  // 2ms
    private let releaseTime: Float = 0.1   // 100ms
    
    func processAudio(_ input: [Float]) -> [Float] {
        var output = [Float](repeating: 0, count: input.count)
        
        for i in 0..<input.count {
            // 1. 현재 레벨 측정
            let currentLevel = abs(input[i])
            
            // 2. 목표 게인 계산
            let targetGain = targetLevel / (currentLevel + 0.0001)
            
            // 3. 부드러운 게인 조정
            if targetGain < currentGain {
                // Attack (빠르게 줄임)
                currentGain += (targetGain - currentGain) * attackTime
            } else {
                // Release (천천히 올림)
                currentGain += (targetGain - currentGain) * releaseTime
            }
            
            // 4. 게인 제한
            currentGain = min(10, max(0.1, currentGain))
            
            // 5. 적용
            output[i] = input[i] * currentGain
            
            // 클리핑 방지
            output[i] = min(1.0, max(-1.0, output[i]))
        }
        
        return output
    }
}
```

## 지터 버퍼: 패킷 순서 정리

```swift
class JitterBuffer {
    private var buffer: [(sequence: Int, timestamp: Int, data: Data)] = []
    private let targetDelay = 40  // ms
    private let maxDelay = 200    // ms
    private var lastPlayedSeq = -1
    
    func addPacket(sequence: Int, timestamp: Int, data: Data) {
        // 중복 체크
        if buffer.contains(where: { $0.sequence == sequence }) {
            return
        }
        
        // 버퍼에 추가
        buffer.append((sequence, timestamp, data))
        
        // 시퀀스 순으로 정렬
        buffer.sort { $0.sequence < $1.sequence }
        
        // 버퍼 크기 제한
        if buffer.count > 10 {
            buffer.removeFirst()
        }
    }
    
    func getNextPacket() -> Data? {
        // 충분한 패킷이 버퍼링될 때까지 대기
        guard buffer.count >= 2 else { return nil }
        
        // 다음 시퀀스 번호 확인
        let expectedSeq = lastPlayedSeq + 1
        
        if let index = buffer.firstIndex(where: { $0.sequence == expectedSeq }) {
            // 정상 순서
            let packet = buffer.remove(at: index)
            lastPlayedSeq = packet.sequence
            return packet.data
        } else if let first = buffer.first,
                  first.sequence > expectedSeq {
            // 패킷 손실 - PLC (Packet Loss Concealment) 필요
            lastPlayedSeq = expectedSeq
            return generateFillerPacket()
        }
        
        return nil
    }
    
    private func generateFillerPacket() -> Data {
        // 이전 패킷 반복 또는 무음 생성
        return Data(repeating: 0, count: 960 * 2)  // 20ms 무음
    }
}
```

## 실시간 오디오 파이프라인

모든 처리를 연결한 완전한 파이프라인:

```swift
class AudioProcessingPipeline {
    // 처리 모듈들
    private let vad = VoiceActivityDetector()
    private let agc = AutomaticGainControl()
    private let noiseReducer = NoiseReducer()
    private let echoCanceller = AcousticEchoCanceller()
    private let opus = OpusProcessor()
    
    // 버퍼
    private let jitterBuffer = JitterBuffer()
    
    // 송신 파이프라인
    func processSendAudio(micInput: [Float], 
                         speakerRef: [Float]) -> Data? {
        // 1. 에코 제거
        let echoFree = echoCanceller.cancelEcho(
            micSignal: micInput,
            speakerSignal: speakerRef
        )
        
        // 2. 노이즈 제거
        let clean = noiseReducer.reduceNoise(in: echoFree)
        
        // 3. AGC
        let normalized = agc.processAudio(clean)
        
        // 4. VAD
        if !vad.detectVoice(in: normalized) {
            return nil  // 무음은 전송 안함
        }
        
        // 5. 인코딩
        let pcm = floatToInt16(normalized)
        return opus.encode(pcm)
    }
    
    // 수신 파이프라인
    func processReceiveAudio(packet: Data, 
                           sequence: Int,
                           timestamp: Int) -> [Float]? {
        // 1. 지터 버퍼
        jitterBuffer.addPacket(sequence: sequence,
                              timestamp: timestamp,
                              data: packet)
        
        guard let data = jitterBuffer.getNextPacket() else {
            return nil
        }
        
        // 2. 디코딩
        guard let pcm = opus.decode(data) else {
            return nil
        }
        
        // 3. Float 변환
        let audio = int16ToFloat(pcm)
        
        // 4. AGC (선택사항)
        return agc.processAudio(audio)
    }
    
    // 헬퍼 함수들
    private func floatToInt16(_ input: [Float]) -> [Int16] {
        return input.map { Int16(max(-32768, min(32767, $0 * 32768))) }
    }
    
    private func int16ToFloat(_ input: [Int16]) -> [Float] {
        return input.map { Float($0) / 32768.0 }
    }
}
```

## 성능 최적화

### SIMD 활용

```swift
import simd

func optimizedDotProduct(_ a: [Float], _ b: [Float]) -> Float {
    let count = a.count
    var result: Float = 0
    
    // SIMD로 4개씩 처리
    let simdCount = count & ~3  // 4의 배수로 내림
    
    for i in stride(from: 0, to: simdCount, by: 4) {
        let va = simd_float4(a[i], a[i+1], a[i+2], a[i+3])
        let vb = simd_float4(b[i], b[i+1], b[i+2], b[i+3])
        result += simd_dot(va, vb)
    }
    
    // 나머지 처리
    for i in simdCount..<count {
        result += a[i] * b[i]
    }
    
    return result
}
```

### Metal로 GPU 가속

```metal
kernel void noiseReduction(
    device const float* input [[buffer(0)]],
    device const float* noiseProfile [[buffer(1)]],
    device float* output [[buffer(2)]],
    uint id [[thread_position_in_grid]]
) {
    float signal = input[id];
    float noise = noiseProfile[id];
    
    // Spectral subtraction
    float clean = max(0.0, signal - noise * 1.5);
    
    // Wiener filter (optional)
    float snr = signal / (noise + 0.001);
    float gain = snr / (snr + 1.0);
    
    output[id] = clean * gain;
}
```

## 음질 측정

### PESQ (Perceptual Evaluation of Speech Quality)

```swift
class AudioQualityMeter {
    func calculatePESQ(reference: [Float], degraded: [Float]) -> Float {
        // ITU-T P.862 알고리즘 (간소화)
        
        // 1. 시간 정렬
        let aligned = alignSignals(reference, degraded)
        
        // 2. 지각 모델 적용
        let refPerceptual = applyPerceptualModel(reference)
        let degPerceptual = applyPerceptualModel(aligned)
        
        // 3. 차이 계산
        let disturbance = calculateDisturbance(refPerceptual, degPerceptual)
        
        // 4. MOS 점수 (1-5)
        return mapToMOS(disturbance)
    }
    
    private func applyPerceptualModel(_ signal: [Float]) -> [Float] {
        // Bark scale 필터뱅크
        // 인간 청각 특성 반영
        return signal  // 간소화
    }
}
```

## 실시간 모니터링

```swift
class AudioMetrics: ObservableObject {
    @Published var inputLevel: Float = 0
    @Published var outputLevel: Float = 0
    @Published var echoReturn: Float = 0
    @Published var noiseLevel: Float = 0
    @Published var packetLoss: Float = 0
    
    func updateMetrics(from pipeline: AudioProcessingPipeline) {
        // UI 업데이트는 메인 스레드에서
        DispatchQueue.main.async {
            self.inputLevel = pipeline.currentInputLevel
            self.outputLevel = pipeline.currentOutputLevel
            // ...
        }
    }
}

// SwiftUI 뷰
struct AudioMetricsView: View {
    @ObservedObject var metrics: AudioMetrics
    
    var body: some View {
        VStack {
            LevelMeter(level: metrics.inputLevel, label: "Input")
            LevelMeter(level: metrics.outputLevel, label: "Output")
            
            HStack {
                MetricBadge(value: metrics.echoReturn, 
                          label: "Echo",
                          unit: "dB")
                MetricBadge(value: metrics.packetLoss, 
                          label: "Loss",
                          unit: "%")
            }
        }
    }
}
```

## 마치며

음성 처리는 과학이자 예술이다.
수학적 정확성과 인간의 지각을 모두 고려해야 한다.

완벽한 에코 제거, 완벽한 노이즈 제거는 불가능하다.
하지만 "충분히 좋은" 품질은 달성할 수 있다.

다음 레벨에서는 이 모든 데이터를 안전하게 지키는 방법,
보안과 프라이버시를 다룰 것이다.

---

## Connections

→ [[L6_security]] - 보안과 프라이버시
→ [[L7_optimization]] - 성능 최적화

← [[L4_networking]] - 네트워킹으로 돌아가기
← [[index]] - 목차

---

Level: L5
Date: 2025-08-15
Tags: #audio #dsp #codec #opus #echo #noise