# AVFoundation: 멀티미디어의 저수준 제어

## Core Insight
AVFoundation은 iOS의 멀티미디어 심장부다. 카메라, 마이크, 비디오 처리의 모든 저수준 제어를 제공하는 강력하지만 복잡한 프레임워크다.

```swift
// 카메라 세션 설정 - 복잡하지만 완전한 제어
let captureSession = AVCaptureSession()
let videoDevice = AVCaptureDevice.default(.builtInWideAngleCamera, 
                                         for: .video, 
                                         position: .back)

let videoInput = try AVCaptureDeviceInput(device: videoDevice!)
captureSession.addInput(videoInput)

let photoOutput = AVCapturePhotoOutput()
captureSession.addOutput(photoOutput)
```

AVFoundation의 계층 구조:
1. **AVCaptureSession**: 전체 캡처 파이프라인 관리
2. **AVCaptureDevice**: 물리적 카메라/마이크
3. **AVCaptureInput**: 입력 소스
4. **AVCaptureOutput**: 출력 형태 (사진, 비디오, 메타데이터)

가장 강력한 기능은 **실시간 처리**다:
```swift
// 실시간 비디오 필터링
let videoDataOutput = AVCaptureVideoDataOutput()
videoDataOutput.setSampleBufferDelegate(self, queue: videoDataOutputQueue)

// 매 프레임마다 호출됨
func captureOutput(_ output: AVCaptureOutput, 
                  didOutput sampleBuffer: CMSampleBuffer, 
                  from connection: AVCaptureConnection) {
    // Core Image, Metal로 실시간 처리 가능
}
```

현대적 기능들:
- **Portrait Mode**: 깊이 정보를 활용한 보케 효과
- **Night Mode**: 저조도에서 자동 장노출
- **Cinematic Mode**: 동영상에서 초점 이동
- **ProRAW**: 전문가급 RAW 이미지

성능 최적화의 핵심:
1. **적절한 session preset**: 용도에 맞는 해상도 선택
2. **Background queue**: 메인 스레드 블록 방지
3. **Memory management**: 대용량 이미지 데이터 관리
4. **Battery optimization**: 불필요한 기능 비활성화

현실: 대부분의 앱은 UIImagePickerController로 충분하다. AVFoundation은 정말 필요할 때만.

## Connections
→ [[271-core-image-processing]]
→ [[272-metal-performance-shaders]]
→ [[273-photo-library-management]]
← [[150-core-ml-on-device-philosophy]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #avfoundation #camera #video #multimedia #performance