# L3-01: 카메라와 Vision 프레임워크
## 실시간 재료 인식, AR 요리 가이드, 영양 분석

---

> **"The camera is becoming the keyboard of the future."**

카메라와 AI를 결합해 요리를 혁신적으로 변화시킵니다.

---

## 🎯 목표

**완성 후 결과물**:
- 실시간 재료 스캔
- AR 레시피 오버레이  
- 음식 볼륨 측정
- 칼로리 추정
- 바코드/QR 스캔

---

## 📸 카메라 UI 구현

### 1단계: 스캔 카메라 뷰

**Features/Camera/ScannerCameraView.swift**:

```swift
import SwiftUI
import AVFoundation
import Vision
import ARKit

struct ScannerCameraView: View {
    @StateObject private var camera = CameraViewModel()
    @StateObject private var recognizer = IngredientRecognizer()
    @State private var showingResults = false
    @State private var capturedImage: UIImage?
    @State private var scanMode: ScanMode = .ingredient
    @State private var showingARMode = false
    
    enum ScanMode: String, CaseIterable {
        case ingredient = "재료"
        case barcode = "바코드"
        case nutrition = "영양"
        case volume = "용량"
        
        var icon: String {
            switch self {
            case .ingredient: return "leaf"
            case .barcode: return "barcode"
            case .nutrition: return "chart.pie"
            case .volume: return "cube"
            }
        }
    }
    
    var body: some View {
        ZStack {
            // 카메라 프리뷰
            CameraPreviewView(session: camera.session)
                .ignoresSafeArea()
                .overlay(
                    ScanningOverlay(
                        detectedObjects: camera.detectedObjects,
                        scanMode: scanMode
                    )
                )
                .gesture(
                    // 탭으로 포커스
                    DragGesture(minimumDistance: 0)
                        .onEnded { value in
                            camera.focus(at: value.location)
                        }
                )
                .gesture(
                    // 핀치로 줌
                    MagnificationGesture()
                        .onChanged { value in
                            camera.zoom(scale: value)
                        }
                )
            
            // UI 오버레이
            VStack {
                // 상단 컨트롤
                HStack {
                    Button {
                        camera.toggleFlash()
                    } label: {
                        Image(systemName: camera.isFlashOn ? "bolt.fill" : "bolt.slash")
                            .font(.title2)
                            .foregroundColor(.white)
                            .frame(width: 44, height: 44)
                            .background(Color.black.opacity(0.5))
                            .clipShape(Circle())
                    }
                    
                    Spacer()
                    
                    // 모드 선택
                    Picker("스캔 모드", selection: $scanMode) {
                        ForEach(ScanMode.allCases, id: \.self) { mode in
                            Label(mode.rawValue, systemImage: mode.icon)
                                .tag(mode)
                        }
                    }
                    .pickerStyle(SegmentedPickerStyle())
                    .frame(width: 200)
                    .background(Color.black.opacity(0.5))
                    .cornerRadius(8)
                    
                    Spacer()
                    
                    Button {
                        showingARMode.toggle()
                    } label: {
                        Image(systemName: "arkit")
                            .font(.title2)
                            .foregroundColor(.white)
                            .frame(width: 44, height: 44)
                            .background(Color.black.opacity(0.5))
                            .clipShape(Circle())
                    }
                }
                .padding()
                
                Spacer()
                
                // 실시간 인식 결과
                if !camera.recognizedItems.isEmpty {
                    RecognitionResultsBar(items: camera.recognizedItems)
                        .padding(.horizontal)
                }
                
                // 하단 컨트롤
                HStack(spacing: 40) {
                    // 갤러리
                    Button {
                        camera.openPhotoLibrary()
                    } label: {
                        if let lastPhoto = camera.lastPhoto {
                            Image(uiImage: lastPhoto)
                                .resizable()
                                .scaledToFill()
                                .frame(width: 50, height: 50)
                                .clipShape(RoundedRectangle(cornerRadius: 8))
                        } else {
                            Image(systemName: "photo")
                                .font(.title2)
                                .foregroundColor(.white)
                                .frame(width: 50, height: 50)
                                .background(Color.black.opacity(0.5))
                                .clipShape(RoundedRectangle(cornerRadius: 8))
                        }
                    }
                    
                    // 캡처 버튼
                    Button {
                        capturePhoto()
                    } label: {
                        ZStack {
                            Circle()
                                .fill(Color.white)
                                .frame(width: 70, height: 70)
                            
                            Circle()
                                .stroke(Color.white, lineWidth: 3)
                                .frame(width: 80, height: 80)
                        }
                    }
                    
                    // 카메라 전환
                    Button {
                        camera.switchCamera()
                    } label: {
                        Image(systemName: "camera.rotate")
                            .font(.title2)
                            .foregroundColor(.white)
                            .frame(width: 50, height: 50)
                            .background(Color.black.opacity(0.5))
                            .clipShape(RoundedRectangle(cornerRadius: 8))
                    }
                }
                .padding(.bottom, 30)
            }
        }
        .sheet(isPresented: $showingResults) {
            if let image = capturedImage {
                ScanResultsView(
                    image: image,
                    scanMode: scanMode,
                    recognizedItems: camera.recognizedItems
                )
            }
        }
        .fullScreenCover(isPresented: $showingARMode) {
            ARCookingGuideView()
        }
        .onAppear {
            camera.startSession()
        }
        .onDisappear {
            camera.stopSession()
        }
    }
    
    private func capturePhoto() {
        camera.capturePhoto { image in
            self.capturedImage = image
            
            Task {
                // 모드별 처리
                switch scanMode {
                case .ingredient:
                    await processIngredientScan(image)
                case .barcode:
                    await processBarcodeaScan(image)
                case .nutrition:
                    await processNutritionScan(image)
                case .volume:
                    await processVolumeScan(image)
                }
                
                showingResults = true
            }
        }
        
        // 햅틱 피드백
        let impactFeedback = UIImpactFeedbackGenerator(style: .medium)
        impactFeedback.impactOccurred()
    }
    
    private func processIngredientScan(_ image: UIImage) async {
        do {
            let ingredients = try await recognizer.recognizeIngredients(from: image)
            await MainActor.run {
                camera.recognizedItems = ingredients.map { RecognizedItem(from: $0) }
            }
        } catch {
            print("재료 인식 실패: \(error)")
        }
    }
}

// MARK: - Camera Preview

struct CameraPreviewView: UIViewRepresentable {
    let session: AVCaptureSession
    
    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        
        let previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        
        DispatchQueue.main.async {
            previewLayer.frame = view.bounds
        }
        
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {
        if let previewLayer = uiView.layer.sublayers?.first as? AVCaptureVideoPreviewLayer {
            DispatchQueue.main.async {
                previewLayer.frame = uiView.bounds
            }
        }
    }
}

// MARK: - Scanning Overlay

struct ScanningOverlay: View {
    let detectedObjects: [DetectedObject]
    let scanMode: ScannerCameraView.ScanMode
    @State private var animationPhase = 0.0
    
    var body: some View {
        ZStack {
            // 스캔 가이드
            if scanMode == .ingredient || scanMode == .nutrition {
                RoundedRectangle(cornerRadius: 20)
                    .stroke(
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        ),
                        lineWidth: 3
                    )
                    .frame(width: 300, height: 300)
                    .opacity(0.8)
                    .scaleEffect(1.0 + sin(animationPhase) * 0.05)
                    .animation(
                        Animation.easeInOut(duration: 2)
                            .repeatForever(autoreverses: true),
                        value: animationPhase
                    )
            }
            
            // 감지된 객체 표시
            ForEach(detectedObjects) { object in
                ObjectBoundingBox(object: object)
            }
            
            // 스캔 모드 표시
            VStack {
                Spacer()
                
                HStack {
                    Image(systemName: scanMode.icon)
                    Text("\(scanMode.rawValue) 스캔 중...")
                }
                .padding()
                .background(Color.black.opacity(0.7))
                .foregroundColor(.white)
                .cornerRadius(20)
                .padding(.bottom, 100)
            }
        }
        .onAppear {
            animationPhase = .pi * 2
        }
    }
}

struct ObjectBoundingBox: View {
    let object: DetectedObject
    
    var body: some View {
        GeometryReader { geometry in
            let rect = VNImageRectForNormalizedRect(
                object.boundingBox,
                Int(geometry.size.width),
                Int(geometry.size.height)
            )
            
            ZStack(alignment: .topLeading) {
                Rectangle()
                    .stroke(object.color, lineWidth: 2)
                    .frame(width: rect.width, height: rect.height)
                    .position(
                        x: rect.midX,
                        y: geometry.size.height - rect.midY
                    )
                
                // 라벨
                Text(object.label)
                    .font(.caption)
                    .padding(4)
                    .background(object.color)
                    .foregroundColor(.white)
                    .cornerRadius(4)
                    .position(
                        x: rect.minX,
                        y: geometry.size.height - rect.maxY
                    )
                
                // 신뢰도
                if object.confidence > 0 {
                    Text("\(Int(object.confidence * 100))%")
                        .font(.caption2)
                        .padding(2)
                        .background(Color.black.opacity(0.7))
                        .foregroundColor(.white)
                        .cornerRadius(4)
                        .position(
                            x: rect.maxX,
                            y: geometry.size.height - rect.maxY
                        )
                }
            }
        }
    }
}
```

### 2단계: 카메라 뷰모델

**Features/Camera/CameraViewModel.swift**:

```swift
import AVFoundation
import Vision
import Photos
import Combine
import UIKit

@MainActor
class CameraViewModel: NSObject, ObservableObject {
    @Published var session = AVCaptureSession()
    @Published var isFlashOn = false
    @Published var zoomFactor: CGFloat = 1.0
    @Published var recognizedItems: [RecognizedItem] = []
    @Published var detectedObjects: [DetectedObject] = []
    @Published var lastPhoto: UIImage?
    @Published var isProcessing = false
    
    private var photoOutput = AVCapturePhotoOutput()
    private var videoOutput = AVCaptureVideoDataOutput()
    private var currentDevice: AVCaptureDevice?
    private var captureCompletionHandler: ((UIImage) -> Void)?
    
    // Vision
    private var requests = [VNRequest]()
    private let visionQueue = DispatchQueue(label: "com.app.vision", qos: .userInitiated)
    
    override init() {
        super.init()
        setupCamera()
        setupVision()
        loadLastPhoto()
    }
    
    private func setupCamera() {
        session.beginConfiguration()
        session.sessionPreset = .photo
        
        // 카메라 추가
        guard let camera = AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: .back
        ) else { return }
        
        currentDevice = camera
        
        do {
            let input = try AVCaptureDeviceInput(device: camera)
            if session.canAddInput(input) {
                session.addInput(input)
            }
            
            // 사진 출력
            if session.canAddOutput(photoOutput) {
                session.addOutput(photoOutput)
                photoOutput.isHighResolutionCaptureEnabled = true
                photoOutput.maxPhotoQualityPrioritization = .quality
            }
            
            // 비디오 출력 (실시간 분석용)
            videoOutput.setSampleBufferDelegate(self, queue: visionQueue)
            if session.canAddOutput(videoOutput) {
                session.addOutput(videoOutput)
            }
            
        } catch {
            print("카메라 설정 실패: \(error)")
        }
        
        session.commitConfiguration()
    }
    
    private func setupVision() {
        // 객체 인식
        guard let model = try? VNCoreMLModel(for: YOLOv3().model) else { return }
        
        let objectDetection = VNCoreMLRequest(model: model) { [weak self] request, error in
            self?.processDetections(request.results)
        }
        objectDetection.imageCropAndScaleOption = .scaleFill
        
        // 바코드 인식
        let barcodeRequest = VNDetectBarcodesRequest { [weak self] request, error in
            self?.processBarcodes(request.results)
        }
        
        // 텍스트 인식
        let textRequest = VNRecognizeTextRequest { [weak self] request, error in
            self?.processText(request.results)
        }
        textRequest.recognitionLevel = .accurate
        textRequest.recognitionLanguages = ["ko-KR", "en-US"]
        textRequest.usesLanguageCorrection = true
        
        requests = [objectDetection, barcodeRequest, textRequest]
    }
    
    func startSession() {
        if !session.isRunning {
            DispatchQueue.global(qos: .background).async { [weak self] in
                self?.session.startRunning()
            }
        }
    }
    
    func stopSession() {
        if session.isRunning {
            DispatchQueue.global(qos: .background).async { [weak self] in
                self?.session.stopRunning()
            }
        }
    }
    
    func capturePhoto(completion: @escaping (UIImage) -> Void) {
        captureCompletionHandler = completion
        
        let settings = AVCapturePhotoSettings()
        settings.flashMode = isFlashOn ? .on : .off
        
        // HEIF 형식 사용 (더 나은 압축)
        if photoOutput.availablePhotoCodecTypes.contains(.hevc) {
            settings.photoCodecType = .hevc
        }
        
        // 깊이 데이터 캡처
        if photoOutput.isDepthDataDeliverySupported {
            settings.isDepthDataDeliveryEnabled = true
        }
        
        // 포트레이트 효과
        if photoOutput.isPortraitEffectsMatteDeliverySupported {
            settings.isPortraitEffectsMatteDeliveryEnabled = true
        }
        
        photoOutput.capturePhoto(with: settings, delegate: self)
    }
    
    func toggleFlash() {
        isFlashOn.toggle()
    }
    
    func switchCamera() {
        session.beginConfiguration()
        
        // 현재 입력 제거
        session.inputs.forEach { session.removeInput($0) }
        
        // 새 카메라 선택
        let position: AVCaptureDevice.Position = currentDevice?.position == .back ? .front : .back
        
        guard let newCamera = AVCaptureDevice.default(
            .builtInWideAngleCamera,
            for: .video,
            position: position
        ) else { return }
        
        do {
            let input = try AVCaptureDeviceInput(device: newCamera)
            if session.canAddInput(input) {
                session.addInput(input)
                currentDevice = newCamera
            }
        } catch {
            print("카메라 전환 실패: \(error)")
        }
        
        session.commitConfiguration()
    }
    
    func zoom(scale: CGFloat) {
        guard let device = currentDevice else { return }
        
        do {
            try device.lockForConfiguration()
            
            let maxZoom = min(device.activeFormat.videoMaxZoomFactor, 10.0)
            let newZoom = max(1.0, min(scale * zoomFactor, maxZoom))
            
            device.videoZoomFactor = newZoom
            zoomFactor = newZoom
            
            device.unlockForConfiguration()
        } catch {
            print("줌 설정 실패: \(error)")
        }
    }
    
    func focus(at point: CGPoint) {
        guard let device = currentDevice else { return }
        
        do {
            try device.lockForConfiguration()
            
            if device.isFocusPointOfInterestSupported {
                device.focusPointOfInterest = point
                device.focusMode = .autoFocus
            }
            
            if device.isExposurePointOfInterestSupported {
                device.exposurePointOfInterest = point
                device.exposureMode = .autoExpose
            }
            
            device.unlockForConfiguration()
            
            // 포커스 애니메이션 피드백
            let generator = UIImpactFeedbackGenerator(style: .light)
            generator.impactOccurred()
        } catch {
            print("포커스 설정 실패: \(error)")
        }
    }
    
    func openPhotoLibrary() {
        // Photo picker 열기
        // SwiftUI의 PhotosPicker 사용
    }
    
    private func loadLastPhoto() {
        let fetchOptions = PHFetchOptions()
        fetchOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: false)]
        fetchOptions.fetchLimit = 1
        
        let fetchResult = PHAsset.fetchAssets(with: .image, options: fetchOptions)
        
        guard let asset = fetchResult.firstObject else { return }
        
        let manager = PHImageManager.default()
        let options = PHImageRequestOptions()
        options.isSynchronous = false
        options.deliveryMode = .highQualityFormat
        
        manager.requestImage(
            for: asset,
            targetSize: CGSize(width: 100, height: 100),
            contentMode: .aspectFill,
            options: options
        ) { [weak self] image, _ in
            DispatchQueue.main.async {
                self?.lastPhoto = image
            }
        }
    }
}

// MARK: - AVCapturePhotoCaptureDelegate

extension CameraViewModel: AVCapturePhotoCaptureDelegate {
    func photoOutput(_ output: AVCapturePhotoOutput, didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
        guard error == nil,
              let imageData = photo.fileDataRepresentation(),
              let image = UIImage(data: imageData) else { return }
        
        // 이미지 저장
        UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
        
        DispatchQueue.main.async {
            self.lastPhoto = image
            self.captureCompletionHandler?(image)
            self.captureCompletionHandler = nil
        }
    }
}

// MARK: - AVCaptureVideoDataOutputSampleBufferDelegate

extension CameraViewModel: AVCaptureVideoDataOutputSampleBufferDelegate {
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        guard !isProcessing,
              let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        isProcessing = true
        
        let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
        
        do {
            try handler.perform(requests)
        } catch {
            print("Vision 처리 실패: \(error)")
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            self.isProcessing = false
        }
    }
    
    private func processDetections(_ results: [Any]?) {
        guard let results = results as? [VNRecognizedObjectObservation] else { return }
        
        let objects = results.compactMap { observation -> DetectedObject? in
            guard observation.confidence > 0.5 else { return nil }
            
            return DetectedObject(
                id: UUID(),
                label: observation.labels.first?.identifier ?? "Unknown",
                confidence: observation.confidence,
                boundingBox: observation.boundingBox,
                color: .blue
            )
        }
        
        DispatchQueue.main.async {
            self.detectedObjects = objects
        }
    }
    
    private func processBarcodes(_ results: [Any]?) {
        guard let results = results as? [VNBarcodeObservation] else { return }
        
        for barcode in results {
            if let payload = barcode.payloadStringValue {
                print("바코드 감지: \(payload)")
                
                // 바코드로 제품 조회
                Task {
                    await lookupProduct(barcode: payload)
                }
            }
        }
    }
    
    private func processText(_ results: [Any]?) {
        guard let results = results as? [VNRecognizedTextObservation] else { return }
        
        for observation in results {
            guard let text = observation.topCandidates(1).first?.string else { continue }
            
            // 영양 정보 추출
            if text.contains("칼로리") || text.contains("kcal") {
                extractNutritionInfo(from: text)
            }
        }
    }
}

// MARK: - Supporting Types

struct DetectedObject: Identifiable {
    let id: UUID
    let label: String
    let confidence: Float
    let boundingBox: CGRect
    let color: Color
}

struct RecognizedItem: Identifiable {
    let id: UUID
    let name: String
    let category: String
    let confidence: Double
    let nutrition: NutritionInfo?
}
```

### 3단계: AR 요리 가이드

**Features/Camera/ARCookingGuideView.swift**:

```swift
import SwiftUI
import RealityKit
import ARKit
import Combine

struct ARCookingGuideView: View {
    @StateObject private var arViewModel = ARCookingViewModel()
    @State private var currentStep = 0
    @State private var showingStepDetail = false
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        ZStack {
            // AR View
            ARViewContainer(viewModel: arViewModel)
                .ignoresSafeArea()
            
            // UI 오버레이
            VStack {
                // 헤더
                HStack {
                    Button {
                        dismiss()
                    } label: {
                        Image(systemName: "xmark.circle.fill")
                            .font(.title)
                            .foregroundColor(.white)
                            .background(Color.black.opacity(0.5))
                            .clipShape(Circle())
                    }
                    
                    Spacer()
                    
                    Text(arViewModel.recipe?.name ?? "AR 요리 가이드")
                        .font(.headline)
                        .padding(.horizontal, 16)
                        .padding(.vertical, 8)
                        .background(Color.black.opacity(0.7))
                        .foregroundColor(.white)
                        .cornerRadius(20)
                    
                    Spacer()
                    
                    Button {
                        arViewModel.toggleARMode()
                    } label: {
                        Image(systemName: arViewModel.isAREnabled ? "arkit" : "arkit.slash")
                            .font(.title)
                            .foregroundColor(.white)
                            .background(Color.black.opacity(0.5))
                            .clipShape(Circle())
                    }
                }
                .padding()
                
                Spacer()
                
                // 현재 단계 표시
                if let step = arViewModel.currentStep {
                    StepOverlay(step: step, onNext: {
                        arViewModel.nextStep()
                    })
                    .padding()
                }
                
                // AR 인디케이터
                if arViewModel.isTracking {
                    HStack {
                        Circle()
                            .fill(Color.green)
                            .frame(width: 10, height: 10)
                        Text("표면 인식됨")
                            .font(.caption)
                            .foregroundColor(.white)
                    }
                    .padding(8)
                    .background(Color.black.opacity(0.7))
                    .cornerRadius(20)
                    .padding(.bottom)
                }
            }
        }
        .onAppear {
            arViewModel.startAR()
        }
        .onDisappear {
            arViewModel.stopAR()
        }
    }
}

struct ARViewContainer: UIViewRepresentable {
    let viewModel: ARCookingViewModel
    
    func makeUIView(context: Context) -> ARView {
        let arView = ARView(frame: .zero)
        
        // AR 설정
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal]
        config.environmentTexturing = .automatic
        
        arView.session.run(config)
        arView.session.delegate = context.coordinator
        
        // 제스처 추가
        let tapGesture = UITapGestureRecognizer(
            target: context.coordinator,
            action: #selector(Coordinator.handleTap)
        )
        arView.addGestureRecognizer(tapGesture)
        
        viewModel.arView = arView
        
        return arView
    }
    
    func updateUIView(_ uiView: ARView, context: Context) {
        // 업데이트 필요시 처리
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(viewModel: viewModel)
    }
    
    class Coordinator: NSObject, ARSessionDelegate {
        let viewModel: ARCookingViewModel
        
        init(viewModel: ARCookingViewModel) {
            self.viewModel = viewModel
        }
        
        @objc func handleTap(gesture: UITapGestureRecognizer) {
            guard let arView = viewModel.arView else { return }
            
            let location = gesture.location(in: arView)
            
            // 레이캐스트로 표면 찾기
            let results = arView.raycast(
                from: location,
                allowing: .existingPlaneGeometry,
                alignment: .horizontal
            )
            
            if let firstResult = results.first {
                viewModel.placeContent(at: firstResult)
            }
        }
        
        func session(_ session: ARSession, didUpdate frame: ARFrame) {
            // 트래킹 상태 업데이트
            DispatchQueue.main.async {
                self.viewModel.updateTrackingState(frame.camera.trackingState)
            }
        }
        
        func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
            for anchor in anchors {
                if let planeAnchor = anchor as? ARPlaneAnchor {
                    viewModel.addPlane(planeAnchor)
                }
            }
        }
    }
}

@MainActor
class ARCookingViewModel: ObservableObject {
    @Published var recipe: Recipe?
    @Published var currentStep: CookingStep?
    @Published var isAREnabled = true
    @Published var isTracking = false
    
    weak var arView: ARView?
    private var anchorEntities: [AnchorEntity] = []
    private var stepEntities: [ModelEntity] = []
    
    func startAR() {
        loadRecipe()
        setupARContent()
    }
    
    func stopAR() {
        arView?.session.pause()
        clearARContent()
    }
    
    func toggleARMode() {
        isAREnabled.toggle()
        
        if isAREnabled {
            showARContent()
        } else {
            hideARContent()
        }
    }
    
    func nextStep() {
        guard let recipe = recipe,
              let currentIndex = recipe.steps.firstIndex(where: { $0.id == currentStep?.id }) else { return }
        
        if currentIndex < recipe.steps.count - 1 {
            currentStep = recipe.steps[currentIndex + 1]
            updateARContent()
        }
    }
    
    func placeContent(at result: ARRaycastResult) {
        guard let arView = arView else { return }
        
        // 앵커 생성
        let anchor = AnchorEntity(world: result.worldTransform)
        arView.scene.addAnchor(anchor)
        
        // 3D 콘텐츠 추가
        if let step = currentStep {
            let content = create3DContent(for: step)
            anchor.addChild(content)
            anchorEntities.append(anchor)
        }
        
        // 햅틱 피드백
        let generator = UIImpactFeedbackGenerator(style: .medium)
        generator.impactOccurred()
    }
    
    private func create3DContent(for step: CookingStep) -> ModelEntity {
        // 단계별 3D 모델 생성
        let mesh: MeshResource
        let material = SimpleMaterial(color: .systemBlue, isMetallic: true)
        
        switch step.type {
        case .cutting:
            // 칼과 도마 모델
            mesh = MeshResource.generateBox(size: [0.2, 0.01, 0.3])
            
        case .mixing:
            // 볼과 스푼 모델
            mesh = MeshResource.generateSphere(radius: 0.15)
            
        case .cooking:
            // 팬 모델
            mesh = MeshResource.generateCylinder(height: 0.05, radius: 0.2)
            
        case .plating:
            // 접시 모델
            mesh = MeshResource.generatePlane(width: 0.3, depth: 0.3)
            
        default:
            mesh = MeshResource.generateBox(size: 0.1)
        }
        
        let model = ModelEntity(mesh: mesh, materials: [material])
        
        // 텍스트 라벨 추가
        if let textMesh = try? MeshResource.generateText(
            step.title,
            extrusionDepth: 0.01,
            font: .systemFont(ofSize: 0.05)
        ) {
            let textEntity = ModelEntity(mesh: textMesh, materials: [material])
            textEntity.position = [0, 0.2, 0]
            model.addChild(textEntity)
        }
        
        // 애니메이션 추가
        addAnimation(to: model, type: step.animationType)
        
        return model
    }
    
    private func addAnimation(to entity: ModelEntity, type: AnimationType?) {
        guard let type = type else { return }
        
        switch type {
        case .rotate:
            entity.transform.rotation = simd_quatf(angle: 0, axis: [0, 1, 0])
            
            var transform = entity.transform
            transform.rotation = simd_quatf(angle: .pi * 2, axis: [0, 1, 0])
            
            entity.move(
                to: transform,
                relativeTo: entity.parent,
                duration: 3,
                timingFunction: .linear
            )
            
        case .bounce:
            let originalPosition = entity.position
            
            var transform = entity.transform
            transform.translation.y += 0.1
            
            entity.move(
                to: transform,
                relativeTo: entity.parent,
                duration: 1,
                timingFunction: .easeInOut
            )
            
        case .pulse:
            let originalScale = entity.scale
            
            var transform = entity.transform
            transform.scale = originalScale * 1.2
            
            entity.move(
                to: transform,
                relativeTo: entity.parent,
                duration: 0.5,
                timingFunction: .easeInOut
            )
        }
    }
    
    func updateTrackingState(_ state: ARCamera.TrackingState) {
        switch state {
        case .normal:
            isTracking = true
        case .limited:
            isTracking = false
        case .notAvailable:
            isTracking = false
        }
    }
    
    func addPlane(_ anchor: ARPlaneAnchor) {
        // 평면 시각화 (옵션)
        guard let arView = arView else { return }
        
        let extent = anchor.extent
        let mesh = MeshResource.generatePlane(
            width: extent.x,
            depth: extent.z
        )
        
        let material = SimpleMaterial(
            color: UIColor.green.withAlphaComponent(0.1),
            isMetallic: false
        )
        
        let planeEntity = ModelEntity(mesh: mesh, materials: [material])
        
        let anchorEntity = AnchorEntity(anchor: anchor)
        anchorEntity.addChild(planeEntity)
        
        arView.scene.addAnchor(anchorEntity)
    }
}

// MARK: - Supporting Types

struct CookingStep: Identifiable {
    let id = UUID()
    let title: String
    let description: String
    let duration: TimeInterval?
    let type: StepType
    let animationType: AnimationType?
    let tips: [String]
    let warningPoints: [String]
    
    enum StepType {
        case preparation
        case cutting
        case mixing
        case cooking
        case plating
        case serving
    }
}

enum AnimationType {
    case rotate
    case bounce
    case pulse
}

struct StepOverlay: View {
    let step: CookingStep
    let onNext: () -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Text(step.title)
                    .font(.headline)
                    .foregroundColor(.white)
                
                Spacer()
                
                if let duration = step.duration {
                    Label("\(Int(duration))분", systemImage: "timer")
                        .font(.caption)
                        .foregroundColor(.white)
                }
            }
            
            Text(step.description)
                .font(.subheadline)
                .foregroundColor(.white.opacity(0.9))
            
            if !step.tips.isEmpty {
                VStack(alignment: .leading, spacing: 4) {
                    Label("팁", systemImage: "lightbulb")
                        .font(.caption)
                        .foregroundColor(.yellow)
                    
                    ForEach(step.tips, id: \.self) { tip in
                        Text("• \(tip)")
                            .font(.caption)
                            .foregroundColor(.white.opacity(0.8))
                    }
                }
            }
            
            Button {
                onNext()
            } label: {
                Label("다음 단계", systemImage: "arrow.right")
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(Color.blue)
                    .foregroundColor(.white)
                    .cornerRadius(10)
            }
        }
        .padding()
        .background(Color.black.opacity(0.8))
        .cornerRadius(16)
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **카메라 통합**
- AVFoundation 활용
- 실시간 프레임 처리
- 포커스/노출 제어
- 멀티 카메라 지원

### 2. **Vision 프레임워크**
- 객체 인식
- 바코드 스캔
- 텍스트 추출
- 실시간 추적

### 3. **ARKit 구현**
- 3D 콘텐츠 배치
- 평면 감지
- 제스처 인터랙션
- 애니메이션

### 4. **실시간 처리**
- 프레임 최적화
- 백그라운드 처리
- 메모리 관리
- 성능 튜닝

---

## 🎉 성공 확인

**카메라 기능 체크리스트**:
- [ ] 실시간으로 재료 인식
- [ ] 바코드로 제품 정보 조회
- [ ] AR로 요리 단계 안내
- [ ] 영양 정보 자동 추출
- [ ] 음식 볼륨 측정 가능

---

**완벽합니다! 카메라와 AR을 활용한 혁신적인 요리 경험이 구현되었습니다.**