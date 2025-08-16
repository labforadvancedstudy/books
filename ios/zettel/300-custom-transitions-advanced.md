# 커스텀 전환 애니메이션: 움직임의 예술

## Core Insight
커스텀 전환은 단순한 시각적 효과를 넘어서, 사용자의 인지적 부담을 줄이고 앱의 공간적 연속성을 구축하는 UX 도구다.

애니메이션은 의미 없는 장식이 아니다. 제대로 설계된 전환 애니메이션은 사용자가 앱의 구조를 이해하고, 현재 위치를 파악하며, 다음에 일어날 일을 예측할 수 있게 도와준다. Apple의 HIG에서 애니메이션을 강조하는 이유가 여기에 있다.

**UIKit 커스텀 전환 아키텍처:**

**1. 내비게이션 컨트롤러 전환:**
```swift
class CustomNavigationAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    enum TransitionType {
        case push
        case pop
    }
    
    private let transitionType: TransitionType
    private let duration: TimeInterval = 0.5
    
    init(type: TransitionType) {
        self.transitionType = type
        super.init()
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return duration
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let fromVC = transitionContext.viewController(forKey: .from),
              let toVC = transitionContext.viewController(forKey: .to) else {
            transitionContext.completeTransition(false)
            return
        }
        
        let containerView = transitionContext.containerView
        
        switch transitionType {
        case .push:
            animatePushTransition(from: fromVC, to: toVC, in: containerView, context: transitionContext)
        case .pop:
            animatePopTransition(from: fromVC, to: toVC, in: containerView, context: transitionContext)
        }
    }
    
    private func animatePushTransition(from fromVC: UIViewController,
                                     to toVC: UIViewController,
                                     in containerView: UIView,
                                     context: UIViewControllerContextTransitioning) {
        // 3D 회전 효과
        containerView.addSubview(toVC.view)
        
        toVC.view.layer.transform = CATransform3DMakeRotation(CGFloat.pi/2, 0, 1, 0)
        fromVC.view.layer.transform = CATransform3DIdentity
        
        UIView.animate(withDuration: duration,
                      delay: 0,
                      usingSpringWithDamping: 0.8,
                      initialSpringVelocity: 0,
                      options: .curveEaseInOut) {
            fromVC.view.layer.transform = CATransform3DMakeRotation(-CGFloat.pi/2, 0, 1, 0)
            toVC.view.layer.transform = CATransform3DIdentity
        } completion: { _ in
            context.completeTransition(!context.transitionWasCancelled)
        }
    }
}

// 네비게이션 컨트롤러에 적용
class CustomNavigationController: UINavigationController {
    override func viewDidLoad() {
        super.viewDidLoad()
        delegate = self
    }
}

extension CustomNavigationController: UINavigationControllerDelegate {
    func navigationController(_ navigationController: UINavigationController,
                            animationControllerFor operation: UINavigationController.Operation,
                            from fromVC: UIViewController,
                            to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        switch operation {
        case .push:
            return CustomNavigationAnimator(type: .push)
        case .pop:
            return CustomNavigationAnimator(type: .pop)
        default:
            return nil
        }
    }
}
```

**2. 인터랙티브 전환:**
```swift
class InteractivePopTransition: UIPercentDrivenInteractiveTransition {
    private weak var navigationController: UINavigationController?
    private var shouldCompleteTransition = false
    private var transitionInProgress = false
    
    init(navigationController: UINavigationController) {
        super.init()
        self.navigationController = navigationController
        setupGestureRecognizer()
    }
    
    private func setupGestureRecognizer() {
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePanGesture(_:)))
        navigationController?.view.addGestureRecognizer(panGesture)
    }
    
    @objc private func handlePanGesture(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: gesture.view)
        let progress = translation.x / (gesture.view?.bounds.width ?? 1)
        
        switch gesture.state {
        case .began:
            transitionInProgress = true
            navigationController?.popViewController(animated: true)
            
        case .changed:
            shouldCompleteTransition = progress > 0.5
            update(progress)
            
        case .cancelled, .ended:
            transitionInProgress = false
            if shouldCompleteTransition {
                finish()
            } else {
                cancel()
            }
            
        default:
            break
        }
    }
}
```

**SwiftUI 커스텀 전환:**

**3. AnyTransition 확장:**
```swift
extension AnyTransition {
    static var flipFromLeft: AnyTransition {
        .asymmetric(
            insertion: .modifier(
                active: FlipModifier(angle: -90, axis: (0, 1, 0)),
                identity: FlipModifier(angle: 0, axis: (0, 1, 0))
            ),
            removal: .modifier(
                active: FlipModifier(angle: 90, axis: (0, 1, 0)),
                identity: FlipModifier(angle: 0, axis: (0, 1, 0))
            )
        )
    }
    
    static var cardStack: AnyTransition {
        .asymmetric(
            insertion: .offset(y: 50).combined(with: .opacity),
            removal: .offset(x: -300).combined(with: .scale(0.8))
        )
    }
    
    static var materialEffect: AnyTransition {
        .asymmetric(
            insertion: .modifier(
                active: MaterialModifier(blur: 10, opacity: 0),
                identity: MaterialModifier(blur: 0, opacity: 1)
            ),
            removal: .modifier(
                active: MaterialModifier(blur: 20, opacity: 0),
                identity: MaterialModifier(blur: 0, opacity: 1)
            )
        )
    }
}

struct FlipModifier: ViewModifier {
    let angle: Double
    let axis: (CGFloat, CGFloat, CGFloat)
    
    func body(content: Content) -> some View {
        content
            .rotation3DEffect(
                .degrees(angle),
                axis: axis,
                perspective: 0.3
            )
    }
}

struct MaterialModifier: ViewModifier {
    let blur: CGFloat
    let opacity: Double
    
    func body(content: Content) -> some View {
        content
            .blur(radius: blur)
            .opacity(opacity)
    }
}
```

**4. 고급 SwiftUI 전환 조합:**
```swift
struct AdvancedTransitionView: View {
    @State private var showDetail = false
    @State private var selectedItem: Item?
    @Namespace private var heroEffect
    
    var body: some View {
        if let item = selectedItem {
            DetailView(item: item, namespace: heroEffect)
                .transition(.asymmetric(
                    insertion: .scale(0.8).combined(with: .opacity),
                    removal: .move(edge: .trailing)
                ))
        } else {
            GridView(selectedItem: $selectedItem, namespace: heroEffect)
        }
    }
}

struct DetailView: View {
    let item: Item
    let namespace: Namespace.ID
    
    var body: some View {
        VStack {
            item.image
                .matchedGeometryEffect(id: "image-\(item.id)", in: namespace)
                .aspectRatio(contentMode: .fit)
            
            Text(item.title)
                .matchedGeometryEffect(id: "title-\(item.id)", in: namespace)
                .font(.largeTitle)
            
            Text(item.description)
                .transition(.opacity.combined(with: .move(edge: .bottom)))
        }
        .navigationBarTitleDisplayMode(.inline)
    }
}
```

**5. 물리 기반 애니메이션:**
```swift
class PhysicsAnimator {
    static func createSpringAnimation(property: String,
                                    toValue: Any,
                                    damping: CGFloat = 0.7,
                                    stiffness: CGFloat = 300) -> CASpringAnimation {
        let spring = CASpringAnimation(keyPath: property)
        spring.toValue = toValue
        spring.damping = damping
        spring.stiffness = stiffness
        spring.mass = 1.0
        spring.initialVelocity = 0
        spring.duration = spring.settlingDuration
        spring.fillMode = .forwards
        spring.isRemovedOnCompletion = false
        
        return spring
    }
    
    static func animateWithPhysics(view: UIView, to position: CGPoint) {
        let positionAnimation = createSpringAnimation(
            property: "position",
            toValue: NSValue(cgPoint: position),
            damping: 15,
            stiffness: 200
        )
        
        view.layer.add(positionAnimation, forKey: "position")
        view.layer.position = position
    }
}
```

**6. 복잡한 시퀀스 애니메이션:**
```swift
class SequenceAnimator {
    static func animateCardFlip(card: UIView, completion: @escaping () -> Void) {
        let duration: TimeInterval = 0.6
        
        UIView.animateKeyframes(withDuration: duration, delay: 0, options: []) {
            // Phase 1: 카드가 중앙으로 회전 (0도 -> 90도)
            UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.5) {
                card.transform = CGAffineTransform(rotationAngle: .pi/2)
                card.transform = card.transform.scaledBy(x: 0.9, y: 0.9)
            }
            
            // Phase 2: 카드 내용 변경 후 완전 회전 (90도 -> 180도)
            UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5) {
                card.transform = CGAffineTransform(rotationAngle: .pi)
                card.transform = card.transform.scaledBy(x: 1.0, y: 1.0)
            }
        } completion: { _ in
            completion()
        }
    }
    
    static func animateWaveEffect(views: [UIView]) {
        for (index, view) in views.enumerated() {
            let delay = Double(index) * 0.1
            
            UIView.animate(withDuration: 0.5,
                          delay: delay,
                          usingSpringWithDamping: 0.7,
                          initialSpringVelocity: 0.5) {
                view.transform = CGAffineTransform(translationX: 0, y: -20)
            } completion: { _ in
                UIView.animate(withDuration: 0.3) {
                    view.transform = .identity
                }
            }
        }
    }
}
```

**7. 성능 최적화된 애니메이션:**
```swift
class OptimizedAnimator {
    static func optimizeForPerformance(layer: CALayer) {
        // 래스터화 활성화 (복잡한 레이어 구조일 때)
        layer.shouldRasterize = true
        layer.rasterizationScale = UIScreen.main.scale
        
        // 애니메이션 후 래스터화 비활성화
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
            layer.shouldRasterize = false
        }
    }
    
    static func animateWithCADisplayLink(view: UIView,
                                       from startValue: CGFloat,
                                       to endValue: CGFloat,
                                       duration: TimeInterval) {
        let displayLink = CADisplayLink(target: self, selector: #selector(updateAnimation))
        displayLink.add(to: .main, forMode: .common)
        
        // 프레임별 정밀 제어가 필요한 애니메이션에 사용
    }
    
    @objc private static func updateAnimation() {
        // 커스텀 애니메이션 로직
    }
}
```

**8. 접근성을 고려한 애니메이션:**
```swift
class AccessibleAnimator {
    static func shouldReduceMotion() -> Bool {
        return UIAccessibility.isReduceMotionEnabled
    }
    
    static func animateRespectingAccessibility(view: UIView,
                                             animations: @escaping () -> Void,
                                             completion: ((Bool) -> Void)? = nil) {
        if shouldReduceMotion() {
            // 모션 감소가 활성화된 경우 즉시 최종 상태로 변경
            animations()
            completion?(true)
        } else {
            // 일반적인 애니메이션 실행
            UIView.animate(withDuration: 0.3,
                          delay: 0,
                          usingSpringWithDamping: 0.8,
                          initialSpringVelocity: 0) {
                animations()
            } completion: { finished in
                completion?(finished)
            }
        }
    }
}
```

커스텀 전환 애니메이션은 앱의 개성을 만드는 중요한 요소다. 하지만 과하면 독이 된다. 사용자의 작업 흐름을 방해하지 않으면서도 의미 있는 피드백을 제공하는 절묘한 균형점을 찾는 것이 핵심이다.

## Connections
→ [[301-metal-high-performance-graphics]]
→ [[302-arkit-lidar-capabilities]]
← [[180-animation-as-communication]]
← [[067-accessibility-as-default]]

---
Level: L3
Date: 2025-08-16
Tags: #animation #transitions #uikit #swiftui #custom-animation #performance #accessibility