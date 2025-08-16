# RTL (Right-to-Left) 언어 지원

## 개념
아랍어, 히브리어 등 오른쪽에서 왼쪽으로 쓰는 언어를 위한 VoIP UI 적응.

## RTL 레이아웃 설정
```swift
class RTLSupport {
    static var isRTL: Bool {
        return UIApplication.shared.userInterfaceLayoutDirection == .rightToLeft
    }
    
    static func setupRTLSupport() {
        // 앱 시작 시 RTL 강제 적용 (테스트용)
        #if DEBUG
        if CommandLine.arguments.contains("-RTL") {
            UIView.appearance().semanticContentAttribute = .forceRightToLeft
        }
        #endif
    }
    
    static func mirrorImageForRTL(_ image: UIImage) -> UIImage {
        if isRTL {
            return image.withHorizontallyFlippedOrientation()
        }
        return image
    }
}
```

## 통화 UI RTL 처리
```swift
class RTLCallViewController: UIViewController {
    @IBOutlet weak var callerImageView: UIImageView!
    @IBOutlet weak var callButtonsStack: UIStackView!
    @IBOutlet weak var leadingConstraint: NSLayoutConstraint!
    @IBOutlet weak var trailingConstraint: NSLayoutConstraint!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupRTLLayout()
    }
    
    private func setupRTLLayout() {
        // 자동 미러링 설정
        view.semanticContentAttribute = .unspecified
        
        // 방향성 아이콘 미러링
        if let backIcon = UIImage(systemName: "arrow.backward") {
            navigationItem.leftBarButtonItem?.image = RTLSupport.mirrorImageForRTL(backIcon)
        }
        
        // 스택 뷰 순서 조정
        if RTLSupport.isRTL {
            callButtonsStack.semanticContentAttribute = .forceRightToLeft
        }
        
        // 제스처 방향 조정
        adjustSwipeGestures()
    }
    
    private func adjustSwipeGestures() {
        // RTL에서 스와이프 방향 반대로
        let swipeGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
        swipeGesture.direction = RTLSupport.isRTL ? .left : .right
        view.addGestureRecognizer(swipeGesture)
    }
}
```

## 텍스트 정렬 RTL
```swift
extension UILabel {
    func setupRTLAlignment() {
        if RTLSupport.isRTL {
            // 자연스러운 텍스트 정렬
            if textAlignment == .left {
                textAlignment = .natural
            }
        }
    }
}

extension UITextField {
    func setupRTLInput() {
        // 텍스트 필드 정렬
        textAlignment = .natural
        
        // 전화번호 입력 특별 처리
        if keyboardType == .phonePad {
            // 전화번호는 항상 LTR
            textAlignment = .left
            semanticContentAttribute = .forceLeftToRight
        }
    }
}
```

## 커스텀 RTL 레이아웃
```swift
class RTLConstraints {
    static func setupRTLConstraints(for view: UIView) {
        // Leading/Trailing 사용 (자동 RTL)
        NSLayoutConstraint.activate([
            view.leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: 16),
            view.trailingAnchor.constraint(equalTo: superview.trailingAnchor, constant: -16)
        ])
        
        // Left/Right 대신 Leading/Trailing 사용
        // 잘못된 예:
        // view.leftAnchor.constraint(equalTo: superview.leftAnchor)
    }
    
    static func mirrorConstraintForRTL(_ constraint: NSLayoutConstraint) -> NSLayoutConstraint {
        if RTLSupport.isRTL {
            // 제약조건 방향 반전
            if constraint.firstAttribute == .leading {
                return NSLayoutConstraint(
                    item: constraint.secondItem as Any,
                    attribute: .trailing,
                    relatedBy: constraint.relation,
                    toItem: constraint.firstItem,
                    attribute: .leading,
                    multiplier: constraint.multiplier,
                    constant: -constraint.constant
                )
            }
        }
        return constraint
    }
}
```

## 애니메이션 RTL
```swift
class RTLAnimations {
    static func slideInAnimation(for view: UIView, completion: (() -> Void)? = nil) {
        let fromX: CGFloat = RTLSupport.isRTL ? -view.frame.width : view.frame.width
        
        view.transform = CGAffineTransform(translationX: fromX, y: 0)
        
        UIView.animate(withDuration: 0.3, animations: {
            view.transform = .identity
        }, completion: { _ in
            completion?()
        })
    }
    
    static func slideOutAnimation(for view: UIView, completion: (() -> Void)? = nil) {
        let toX: CGFloat = RTLSupport.isRTL ? view.frame.width : -view.frame.width
        
        UIView.animate(withDuration: 0.3, animations: {
            view.transform = CGAffineTransform(translationX: toX, y: 0)
        }, completion: { _ in
            completion?()
        })
    }
}
```

## Collection View RTL
```swift
class RTLCollectionViewFlowLayout: UICollectionViewFlowLayout {
    override var flipsHorizontallyInOppositeLayoutDirection: Bool {
        return true  // 자동 RTL 처리
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        guard let attributes = super.layoutAttributesForElements(in: rect) else { return nil }
        
        if RTLSupport.isRTL {
            for attribute in attributes {
                // RTL에서 셀 순서 조정
                let collectionViewWidth = collectionView?.frame.width ?? 0
                attribute.frame.origin.x = collectionViewWidth - attribute.frame.origin.x - attribute.frame.width
            }
        }
        
        return attributes
    }
}
```

## 연관 개념
- [[localization_i18n]]
- [[dynamic_type]]
- [[accessibility_voiceover]]

## 태그
#rtl #localization #arabic #hebrew #ui #layout