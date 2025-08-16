# ARKit과 LiDAR: 공간 인식의 혁명

## Core Insight
ARKit과 LiDAR의 결합은 디지털과 물리적 세계 사이의 경계를 허물며, 스마트폰을 3차원 공간을 이해하는 컴퓨터로 변화시킨다.

LiDAR(Light Detection and Ranging)는 iPhone 12 Pro부터 탑재된 기술로, 공간을 밀리미터 단위로 실시간 스캔할 수 있다. 이는 단순한 카메라 기반 AR을 넘어서, 진정한 공간 컴퓨팅을 가능하게 한다.

**ARKit 6+ 핵심 기능들:**

**1. 기본 ARKit 설정과 LiDAR 활용:**
```swift
import ARKit
import RealityKit

class ARViewController: UIViewController, ARSessionDelegate {
    @IBOutlet var arView: ARView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupARSession()
    }
    
    private func setupARSession() {
        // LiDAR 지원 확인
        guard ARWorldTrackingConfiguration.supportsSceneReconstruction(.mesh) else {
            print("LiDAR not supported on this device")
            return
        }
        
        let configuration = ARWorldTrackingConfiguration()
        
        // Scene reconstruction 활성화 (LiDAR 필요)
        configuration.sceneReconstruction = .mesh
        
        // 평면 감지
        configuration.planeDetection = [.horizontal, .vertical]
        
        // 환경 텍스처링
        configuration.environmentTexturing = .automatic
        
        // 사람 감지 (iPhone 12+ 필요)
        if ARBodyTrackingConfiguration.isSupported {
            configuration.frameSemantics.insert(.personSegmentationWithDepth)
        }
        
        arView.session.run(configuration)
        arView.session.delegate = self
    }
    
    // MARK: - ARSessionDelegate
    func session(_ session: ARSession, didUpdate frame: ARFrame) {
        // 실시간 깊이 데이터 처리
        if let depthData = frame.sceneDepth {
            processDepthData(depthData)
        }
        
        // 사람 세그멘테이션 처리
        if let segmentationBuffer = frame.segmentationBuffer {
            processPersonSegmentation(segmentationBuffer)
        }
    }
    
    private func processDepthData(_ depthData: ARDepthData) {
        let depthMap = depthData.depthMap
        let confidence = depthData.confidenceMap
        
        // 깊이 데이터를 활용한 정밀한 객체 배치
        let width = CVPixelBufferGetWidth(depthMap)
        let height = CVPixelBufferGetHeight(depthMap)
        
        // 중앙점의 실제 거리 계산
        let centerX = width / 2
        let centerY = height / 2
        
        CVPixelBufferLockBaseAddress(depthMap, .readOnly)
        defer { CVPixelBufferUnlockBaseAddress(depthMap, .readOnly) }
        
        let depthPointer = CVPixelBufferGetBaseAddress(depthMap)?.assumingMemoryBound(to: Float32.self)
        let depthValue = depthPointer?[centerY * width + centerX] ?? 0
        
        print("Center depth: \(depthValue) meters")
    }
}
```

**2. 정밀한 객체 배치와 충돌 감지:**
```swift
class PreciseObjectPlacement {
    private var arView: ARView
    
    init(arView: ARView) {
        self.arView = arView
    }
    
    func placeObjectAt(screenPoint: CGPoint) {
        // 레이캐스팅으로 정확한 3D 위치 찾기
        let raycastQuery = arView.makeRaycastQuery(from: screenPoint,
                                                  allowing: .estimatedPlane,
                                                  alignment: .any)
        
        guard let query = raycastQuery else { return }
        
        let results = arView.session.raycast(query)
        
        if let firstResult = results.first {
            // 높은 정확도의 위치 정보
            let transform = firstResult.worldTransform
            let position = SIMD3<Float>(transform.columns.3.x,
                                       transform.columns.3.y,
                                       transform.columns.3.z)
            
            // 객체 생성 및 배치
            let entity = createVirtualObject()
            entity.transform.translation = position
            
            // 물리 기반 충돌 감지
            entity.components[CollisionComponent.self] = CollisionComponent(
                shapes: [.generateBox(size: [0.1, 0.1, 0.1])],
                mode: .trigger,
                filter: CollisionFilter(group: .default, mask: .all)
            )
            
            arView.scene.addAnchor(entity)
        }
    }
    
    private func createVirtualObject() -> AnchorEntity {
        let anchor = AnchorEntity()
        
        // 3D 모델 로드
        let mesh = MeshResource.generateBox(size: 0.1)
        let material = SimpleMaterial(color: .blue, isMetallic: false)
        let modelComponent = ModelComponent(mesh: mesh, materials: [material])
        
        let entity = Entity()
        entity.components[ModelComponent.self] = modelComponent
        
        anchor.addChild(entity)
        return anchor
    }
}
```

**3. 실시간 3D 스캐닝과 메시 재구성:**
```swift
class SceneReconstruction: ObservableObject {
    @Published var meshAnchors: [ARMeshAnchor] = []
    private var arView: ARView
    
    init(arView: ARView) {
        self.arView = arView
    }
    
    func startScanning() {
        // 메시 재구성 시작
        let configuration = ARWorldTrackingConfiguration()
        configuration.sceneReconstruction = .meshWithClassification
        arView.session.run(configuration)
    }
    
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        for anchor in anchors {
            if let meshAnchor = anchor as? ARMeshAnchor {
                addMeshAnchor(meshAnchor)
            }
        }
    }
    
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            if let meshAnchor = anchor as? ARMeshAnchor {
                updateMeshAnchor(meshAnchor)
            }
        }
    }
    
    private func addMeshAnchor(_ meshAnchor: ARMeshAnchor) {
        let geometry = meshAnchor.geometry
        
        // 버텍스 데이터 추출
        let vertices = geometry.vertices
        let normals = geometry.normals
        let faces = geometry.faces
        
        // RealityKit 메시 생성
        let meshResource = generateMeshResource(vertices: vertices, 
                                              normals: normals, 
                                              faces: faces)
        
        // 와이어프레임 재질로 시각화
        let material = UnlitMaterial(color: .white)
        material.baseColor = MaterialColorParameter.color(.green.withAlphaComponent(0.3))
        
        let modelComponent = ModelComponent(mesh: meshResource, materials: [material])
        
        let entity = Entity()
        entity.components[ModelComponent.self] = modelComponent
        entity.transform = Transform(matrix: meshAnchor.transform)
        
        let anchorEntity = AnchorEntity(anchor: meshAnchor)
        anchorEntity.addChild(entity)
        
        arView.scene.addAnchor(anchorEntity)
        
        DispatchQueue.main.async {
            self.meshAnchors.append(meshAnchor)
        }
    }
    
    private func generateMeshResource(vertices: ARGeometrySource,
                                    normals: ARGeometrySource,
                                    faces: ARGeometryElement) -> MeshResource {
        // 복잡한 메시 생성 로직
        // 실제 구현에서는 Metal 버퍼를 직접 활용
        return MeshResource.generateBox(size: 0.1) // placeholder
    }
}
```

**4. 고급 평면 감지와 분류:**
```swift
class AdvancedPlaneDetection {
    func classifyPlane(_ planeAnchor: ARPlaneAnchor) -> PlaneType {
        let transform = planeAnchor.transform
        let normal = SIMD3<Float>(transform.columns.1.x, transform.columns.1.y, transform.columns.1.z)
        
        // 평면의 방향으로 분류
        let angle = acos(dot(normal, SIMD3<Float>(0, 1, 0)))
        let degrees = angle * 180 / .pi
        
        switch degrees {
        case 0..<15:
            return .horizontal(.floor)
        case 15..<75:
            return .inclined
        case 75..<105:
            return .vertical(.wall)
        case 105..<165:
            return .inclined
        case 165...180:
            return .horizontal(.ceiling)
        default:
            return .unknown
        }
    }
    
    enum PlaneType {
        case horizontal(HorizontalType)
        case vertical(VerticalType)
        case inclined
        case unknown
        
        enum HorizontalType {
            case floor, table, ceiling
        }
        
        enum VerticalType {
            case wall, door, window
        }
    }
    
    func optimizePlaneForOcclusion(_ planeAnchor: ARPlaneAnchor) -> ModelComponent? {
        // 오클루전을 위한 투명 평면 생성
        let mesh = MeshResource.generatePlane(width: planeAnchor.planeExtent.width,
                                            depth: planeAnchor.planeExtent.height)
        
        var material = UnlitMaterial()
        material.color.tint = .clear
        material.triangle.fillMode = .fill
        
        // 오클루전만 수행하고 렌더링하지 않음
        material.baseColor = MaterialColorParameter.color(.clear)
        
        return ModelComponent(mesh: mesh, materials: [material])
    }
}
```

**5. 사람 감지와 상호작용:**
```swift
class PersonInteraction {
    func trackPersonSkeleton(in frame: ARFrame) {
        guard let bodyAnchor = frame.anchors.compactMap({ $0 as? ARBodyAnchor }).first else {
            return
        }
        
        let skeleton = bodyAnchor.skeleton
        
        // 주요 관절 위치 추출
        let jointNames: [ARSkeleton.JointName] = [
            .head, .neck_1,
            .left_shoulder, .right_shoulder,
            .left_hand, .right_hand,
            .left_hip, .right_hip,
            .left_foot, .right_foot
        ]
        
        var jointPositions: [ARSkeleton.JointName: SIMD3<Float>] = [:]
        
        for jointName in jointNames {
            if skeleton.isJointTracked(jointName) {
                let transform = skeleton.modelTransform(for: jointName)
                let position = SIMD3<Float>(transform?.columns.3.x ?? 0,
                                          transform?.columns.3.y ?? 0,
                                          transform?.columns.3.z ?? 0)
                jointPositions[jointName] = position
            }
        }
        
        // 제스처 인식
        recognizeGestures(from: jointPositions)
    }
    
    private func recognizeGestures(from joints: [ARSkeleton.JointName: SIMD3<Float>]) {
        guard let leftHand = joints[.left_hand],
              let rightHand = joints[.right_hand],
              let head = joints[.head] else { return }
        
        // 손 흔들기 제스처 감지
        let handDistance = distance(leftHand, rightHand)
        let headToHandDistance = distance(head, leftHand)
        
        if handDistance < 0.3 && headToHandDistance < 0.5 {
            // 박수 제스처로 인식
            triggerClapAction()
        }
        
        // 포인팅 제스처 감지
        let shoulderToHand = rightHand - (joints[.right_shoulder] ?? SIMD3<Float>(0,0,0))
        if length(shoulderToHand) > 0.4 {
            let pointingDirection = normalize(shoulderToHand)
            handlePointingGesture(direction: pointingDirection, from: rightHand)
        }
    }
    
    private func triggerClapAction() {
        print("Clap gesture detected!")
        // 박수에 반응하는 액션 실행
    }
    
    private func handlePointingGesture(direction: SIMD3<Float>, from position: SIMD3<Float>) {
        print("Pointing gesture detected at direction: \(direction)")
        // 포인팅 방향의 객체와 상호작용
    }
}
```

**6. 실시간 오클루전과 조명:**
```swift
class RealisticRendering {
    func setupRealisticEnvironment(arView: ARView) {
        // 환경 조명 자동 추정
        let configuration = ARWorldTrackingConfiguration()
        configuration.environmentTexturing = .automatic
        
        // 실시간 조명 추정
        if ARWorldTrackingConfiguration.supportsFrameSemantics(.sceneDepth) {
            configuration.frameSemantics.insert(.sceneDepth)
        }
        
        arView.session.run(configuration)
        
        // 환경 조명 설정
        arView.environment.lighting.resource = try? EnvironmentResource.load(named: "studio")
        arView.environment.sceneKit.lightingEnvironment = SCNEnvironment()
    }
    
    func addRealisticShadows(to entity: Entity, in arView: ARView) {
        // 그림자 캐스팅 활성화
        entity.components[ModelComponent.self]?.materials = entity.components[ModelComponent.self]?.materials.map { material in
            var updatedMaterial = material
            if var pbr = updatedMaterial as? PhysicallyBasedMaterial {
                pbr.faceCulling = .none
                pbr.clearcoat = PhysicallyBasedMaterial.Clearcoat(floatLiteral: 0.5)
                return pbr
            }
            return updatedMaterial
        } ?? []
        
        // 오클루전 머티리얼 적용
        let occlusionMaterial = OcclusionMaterial()
        
        // 가상 평면에 오클루전 적용
        if let planeAnchor = arView.scene.anchors.first(where: { $0 is ARPlaneAnchor }) {
            let occlusionEntity = Entity()
            let mesh = MeshResource.generatePlane(width: 2, depth: 2)
            occlusionEntity.components[ModelComponent.self] = ModelComponent(mesh: mesh, materials: [occlusionMaterial])
            planeAnchor.addChild(occlusionEntity)
        }
    }
}
```

**7. 다중 사용자 AR 세션:**
```swift
class CollaborativeAR {
    private var arView: ARView
    private var multipeerSession: MultipeerConnectivityService
    
    init(arView: ARView) {
        self.arView = arView
        self.multipeerSession = MultipeerConnectivityService()
        setupCollaborativeSession()
    }
    
    private func setupCollaborativeSession() {
        let configuration = ARWorldTrackingConfiguration()
        configuration.isCollaborationEnabled = true
        
        arView.session.run(configuration)
        
        // 다른 사용자의 정보 공유
        multipeerSession.onReceiveData = { [weak self] data in
            self?.handleCollaborativeData(data)
        }
    }
    
    func shareAnchor(_ anchor: ARAnchor) {
        guard let data = try? NSKeyedArchiver.archivedData(withRootObject: anchor, requiringSecureCoding: true) else {
            return
        }
        
        multipeerSession.sendToAllPeers(data)
    }
    
    private func handleCollaborativeData(_ data: Data) {
        guard let anchor = try? NSKeyedUnarchiver.unarchivedObject(ofClass: ARAnchor.self, from: data) else {
            return
        }
        
        // 다른 사용자가 생성한 앵커 추가
        arView.session.add(anchor: anchor)
    }
}
```

ARKit과 LiDAR의 결합은 모바일 AR의 가능성을 완전히 새로운 차원으로 끌어올렸다. 정밀한 깊이 인식, 실시간 3D 스캐닝, 사실적인 오클루전은 이제 스마트폰에서도 가능하다. 이는 단순한 기술 발전을 넘어서, 공간 컴퓨팅 시대의 시작을 알린다.

## Connections
→ [[303-homekit-iot-integration]]
→ [[304-carplay-external-display]]
← [[301-metal-high-performance-graphics]]
← [[058-spatial-computing-paradigm]]

---
Level: L2
Date: 2025-08-16
Tags: #arkit #lidar #spatial-computing #3d-scanning #ar #depth-sensing #collaboration