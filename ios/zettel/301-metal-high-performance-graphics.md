# Metal: 고성능 그래픽스의 원시 파워

## Core Insight
Metal은 GPU의 원시적 파워에 직접 접근할 수 있게 해주는 저수준 API로, Core Graphics나 Core Animation으로는 불가능한 실시간 렌더링을 가능케 한다.

Metal은 Apple Silicon의 통합 메모리 아키텍처와 완벽하게 최적화되어 있다. CPU와 GPU가 같은 메모리 풀을 공유하므로, 데이터 복사 없이 직접 GPU 연산이 가능하다. 이는 60fps, 120fps의 실시간 렌더링에서 결정적 차이를 만든다.

**Metal 파이프라인 아키텍처:**

**1. 기본 Metal 설정:**
```swift
import Metal
import MetalKit

class MetalRenderer: NSObject, MTKViewDelegate {
    private var device: MTLDevice!
    private var commandQueue: MTLCommandQueue!
    private var renderPipelineState: MTLRenderPipelineState!
    private var vertexBuffer: MTLBuffer!
    
    override init() {
        super.init()
        setupMetal()
        setupRenderPipeline()
        setupVertexData()
    }
    
    private func setupMetal() {
        // Metal 디바이스 초기화
        guard let device = MTLCreateSystemDefaultDevice() else {
            fatalError("Metal is not supported on this device")
        }
        
        self.device = device
        self.commandQueue = device.makeCommandQueue()
    }
    
    private func setupRenderPipeline() {
        // 셰이더 라이브러리 로드
        guard let library = device.makeDefaultLibrary() else {
            fatalError("Could not create Metal library")
        }
        
        let vertexFunction = library.makeFunction(name: "vertex_main")
        let fragmentFunction = library.makeFunction(name: "fragment_main")
        
        // 렌더 파이프라인 설정
        let pipelineDescriptor = MTLRenderPipelineDescriptor()
        pipelineDescriptor.vertexFunction = vertexFunction
        pipelineDescriptor.fragmentFunction = fragmentFunction
        pipelineDescriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        
        do {
            renderPipelineState = try device.makeRenderPipelineState(descriptor: pipelineDescriptor)
        } catch {
            fatalError("Failed to create render pipeline state: \(error)")
        }
    }
    
    func mtkView(_ view: MTKView, drawableSizeWillChange size: CGSize) {
        // 뷰 크기 변경 시 처리
    }
    
    func draw(in view: MTKView) {
        // 프레임별 렌더링
        guard let drawable = view.currentDrawable,
              let renderPassDescriptor = view.currentRenderPassDescriptor else { return }
        
        let commandBuffer = commandQueue.makeCommandBuffer()!
        let renderEncoder = commandBuffer.makeRenderCommandEncoder(descriptor: renderPassDescriptor)!
        
        renderEncoder.setRenderPipelineState(renderPipelineState)
        renderEncoder.setVertexBuffer(vertexBuffer, offset: 0, index: 0)
        renderEncoder.drawPrimitives(type: .triangle, vertexStart: 0, vertexCount: 3)
        
        renderEncoder.endEncoding()
        commandBuffer.present(drawable)
        commandBuffer.commit()
    }
}
```

**2. 컴퓨트 셰이더를 활용한 병렬 처리:**
```swift
class MetalImageProcessor {
    private var device: MTLDevice!
    private var commandQueue: MTLCommandQueue!
    private var computePipelineState: MTLComputePipelineState!
    
    func setupComputePipeline() {
        device = MTLCreateSystemDefaultDevice()!
        commandQueue = device.makeCommandQueue()!
        
        let library = device.makeDefaultLibrary()!
        let kernelFunction = library.makeFunction(name: "blur_kernel")!
        
        do {
            computePipelineState = try device.makeComputePipelineState(function: kernelFunction)
        } catch {
            fatalError("Failed to create compute pipeline: \(error)")
        }
    }
    
    func processImage(_ inputImage: UIImage) -> UIImage? {
        // 입력 이미지를 Metal 텍스처로 변환
        guard let inputTexture = makeTexture(from: inputImage),
              let outputTexture = makeTexture(width: Int(inputImage.size.width),
                                            height: Int(inputImage.size.height)) else {
            return nil
        }
        
        // 컴퓨트 명령 실행
        let commandBuffer = commandQueue.makeCommandBuffer()!
        let computeEncoder = commandBuffer.makeComputeCommandEncoder()!
        
        computeEncoder.setComputePipelineState(computePipelineState)
        computeEncoder.setTexture(inputTexture, index: 0)
        computeEncoder.setTexture(outputTexture, index: 1)
        
        // 스레드 그룹 설정
        let threadgroupSize = MTLSize(width: 16, height: 16, depth: 1)
        let threadgroupCount = MTLSize(
            width: (inputTexture.width + threadgroupSize.width - 1) / threadgroupSize.width,
            height: (inputTexture.height + threadgroupSize.height - 1) / threadgroupSize.height,
            depth: 1
        )
        
        computeEncoder.dispatchThreadgroups(threadgroupCount, threadsPerThreadgroup: threadgroupSize)
        computeEncoder.endEncoding()
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        // 결과 텍스처를 UIImage로 변환
        return makeImage(from: outputTexture)
    }
}
```

**3. Metal 셰이더 언어 (MSL):**
```metal
// Shaders.metal
#include <metal_stdlib>
using namespace metal;

struct VertexIn {
    float4 position [[attribute(0)]];
    float4 color [[attribute(1)]];
    float2 texCoord [[attribute(2)]];
};

struct VertexOut {
    float4 position [[position]];
    float4 color;
    float2 texCoord;
};

struct Uniforms {
    float4x4 modelViewProjectionMatrix;
    float time;
    float4 lightPosition;
};

// 버텍스 셰이더
vertex VertexOut vertex_main(VertexIn in [[stage_in]],
                            constant Uniforms& uniforms [[buffer(1)]]) {
    VertexOut out;
    out.position = uniforms.modelViewProjectionMatrix * in.position;
    out.color = in.color;
    out.texCoord = in.texCoord;
    
    // 시간 기반 애니메이션
    float wave = sin(uniforms.time + in.position.x * 10.0) * 0.1;
    out.position.y += wave;
    
    return out;
}

// 프래그먼트 셰이더
fragment float4 fragment_main(VertexOut in [[stage_in]],
                             texture2d<float> colorTexture [[texture(0)]],
                             sampler textureSampler [[sampler(0)]],
                             constant Uniforms& uniforms [[buffer(0)]]) {
    float4 texColor = colorTexture.sample(textureSampler, in.texCoord);
    
    // 조명 계산
    float3 lightDir = normalize(uniforms.lightPosition.xyz - in.position.xyz);
    float lightIntensity = max(dot(float3(0, 0, 1), lightDir), 0.1);
    
    return texColor * in.color * lightIntensity;
}

// 컴퓨트 셰이더 - 이미지 블러 효과
kernel void blur_kernel(texture2d<float, access::read> inputTexture [[texture(0)]],
                       texture2d<float, access::write> outputTexture [[texture(1)]],
                       uint2 gid [[thread_position_in_grid]]) {
    if (gid.x >= outputTexture.get_width() || gid.y >= outputTexture.get_height()) {
        return;
    }
    
    float4 color = float4(0.0);
    const int radius = 5;
    float count = 0.0;
    
    // 가우시안 블러 적용
    for (int x = -radius; x <= radius; x++) {
        for (int y = -radius; y <= radius; y++) {
            uint2 coord = uint2(int(gid.x) + x, int(gid.y) + y);
            if (coord.x < inputTexture.get_width() && coord.y < inputTexture.get_height()) {
                color += inputTexture.read(coord);
                count += 1.0;
            }
        }
    }
    
    outputTexture.write(color / count, gid);
}
```

**4. 실시간 필터 체인:**
```swift
class MetalFilterChain {
    private var device: MTLDevice!
    private var commandQueue: MTLCommandQueue!
    private var filters: [MetalFilter] = []
    
    protocol MetalFilter {
        func process(input: MTLTexture, output: MTLTexture, commandBuffer: MTLCommandBuffer)
    }
    
    class BlurFilter: MetalFilter {
        private var computePipelineState: MTLComputePipelineState!
        
        init(device: MTLDevice) {
            setupPipeline(device: device)
        }
        
        func process(input: MTLTexture, output: MTLTexture, commandBuffer: MTLCommandBuffer) {
            let computeEncoder = commandBuffer.makeComputeCommandEncoder()!
            computeEncoder.setComputePipelineState(computePipelineState)
            computeEncoder.setTexture(input, index: 0)
            computeEncoder.setTexture(output, index: 1)
            
            let threadgroupSize = MTLSize(width: 16, height: 16, depth: 1)
            let threadgroupCount = MTLSize(
                width: (input.width + 15) / 16,
                height: (input.height + 15) / 16,
                depth: 1
            )
            
            computeEncoder.dispatchThreadgroups(threadgroupCount, threadsPerThreadgroup: threadgroupSize)
            computeEncoder.endEncoding()
        }
    }
    
    func addFilter(_ filter: MetalFilter) {
        filters.append(filter)
    }
    
    func processImage(_ inputTexture: MTLTexture) -> MTLTexture? {
        var currentTexture = inputTexture
        var intermediateTexture: MTLTexture?
        
        let commandBuffer = commandQueue.makeCommandBuffer()!
        
        for (index, filter) in filters.enumerated() {
            if index == filters.count - 1 {
                // 마지막 필터는 최종 출력 텍스처로
                filter.process(input: currentTexture, output: finalOutputTexture, commandBuffer: commandBuffer)
            } else {
                // 중간 텍스처 생성
                intermediateTexture = createIntermediateTexture(like: currentTexture)!
                filter.process(input: currentTexture, output: intermediateTexture!, commandBuffer: commandBuffer)
                currentTexture = intermediateTexture!
            }
        }
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return currentTexture
    }
}
```

**5. Metal Performance Shaders (MPS) 활용:**
```swift
import MetalPerformanceShaders

class MPSImageProcessor {
    private var device: MTLDevice!
    private var commandQueue: MTLCommandQueue!
    
    func applyGaussianBlur(to texture: MTLTexture, sigma: Float) -> MTLTexture? {
        let blur = MPSImageGaussianBlur(device: device, sigma: sigma)
        
        let outputDescriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: texture.pixelFormat,
            width: texture.width,
            height: texture.height,
            mipmapped: false
        )
        outputDescriptor.usage = [.shaderRead, .shaderWrite]
        
        guard let outputTexture = device.makeTexture(descriptor: outputDescriptor) else {
            return nil
        }
        
        let commandBuffer = commandQueue.makeCommandBuffer()!
        blur.encode(commandBuffer: commandBuffer, sourceTexture: texture, destinationTexture: outputTexture)
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        return outputTexture
    }
    
    func applyConvolution(to texture: MTLTexture, kernel: [Float]) -> MTLTexture? {
        let kernelSize = Int(sqrt(Float(kernel.count)))
        let convolution = MPSImageConvolution(device: device,
                                            kernelWidth: kernelSize,
                                            kernelHeight: kernelSize,
                                            weights: kernel)
        
        // 컨볼루션 적용...
        return nil // placeholder
    }
}
```

**6. 메모리 관리와 성능 최적화:**
```swift
class MetalMemoryManager {
    private var device: MTLDevice!
    private var textureCache: [String: MTLTexture] = [:]
    private var bufferPool: [MTLBuffer] = []
    
    func getReusableTexture(width: Int, height: Int, pixelFormat: MTLPixelFormat) -> MTLTexture? {
        let key = "\(width)x\(height)_\(pixelFormat.rawValue)"
        
        if let cachedTexture = textureCache[key] {
            return cachedTexture
        }
        
        let descriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: pixelFormat,
            width: width,
            height: height,
            mipmapped: false
        )
        descriptor.usage = [.shaderRead, .shaderWrite, .renderTarget]
        
        let texture = device.makeTexture(descriptor: descriptor)
        textureCache[key] = texture
        
        return texture
    }
    
    func getReusableBuffer(length: Int) -> MTLBuffer? {
        // 적절한 크기의 버퍼를 풀에서 찾아 재사용
        for (index, buffer) in bufferPool.enumerated() {
            if buffer.length >= length {
                bufferPool.remove(at: index)
                return buffer
            }
        }
        
        // 새 버퍼 생성
        return device.makeBuffer(length: length, options: .cpuCacheModeWriteCombined)
    }
    
    func returnBuffer(_ buffer: MTLBuffer) {
        bufferPool.append(buffer)
        
        // 풀 크기 제한
        if bufferPool.count > 10 {
            bufferPool.removeFirst()
        }
    }
}
```

**7. Metal과 Core Image 통합:**
```swift
class MetalCoreImageProcessor {
    private var context: CIContext!
    private var device: MTLDevice!
    
    init() {
        device = MTLCreateSystemDefaultDevice()!
        context = CIContext(mtlDevice: device)
    }
    
    func processWithCoreImage(_ inputTexture: MTLTexture) -> MTLTexture? {
        // Metal 텍스처를 CIImage로 변환
        let ciImage = CIImage(mtlTexture: inputTexture, options: nil)!
        
        // Core Image 필터 적용
        let filter = CIFilter.colorControls()
        filter.inputImage = ciImage
        filter.saturation = 1.5
        filter.brightness = 0.1
        filter.contrast = 1.2
        
        guard let outputImage = filter.outputImage else { return nil }
        
        // 다시 Metal 텍스처로 변환
        let outputDescriptor = MTLTextureDescriptor.texture2DDescriptor(
            pixelFormat: inputTexture.pixelFormat,
            width: inputTexture.width,
            height: inputTexture.height,
            mipmapped: false
        )
        outputDescriptor.usage = [.shaderRead, .shaderWrite]
        
        guard let outputTexture = device.makeTexture(descriptor: outputDescriptor) else {
            return nil
        }
        
        context.render(outputImage, to: outputTexture, commandBuffer: nil, bounds: outputImage.extent, colorSpace: CGColorSpaceCreateDeviceRGB())
        
        return outputTexture
    }
}
```

Metal은 iOS에서 가장 강력한 그래픽스 API다. 하지만 그 강력함과 함께 복잡성도 따라온다. Core Animation이나 Core Graphics로 해결되지 않는 성능 문제가 있을 때, Metal은 최후의 해답이 될 수 있다. 특히 실시간 이미지/비디오 처리, 게임, AR/VR 애플리케이션에서 그 진가를 발휘한다.

## Connections
→ [[302-arkit-lidar-capabilities]]
→ [[303-homekit-iot-integration]]
← [[300-custom-transitions-advanced]]
← [[150-core-ml-on-device-philosophy]]

---
Level: L2
Date: 2025-08-16
Tags: #metal #gpu #high-performance #graphics #shaders #compute #optimization