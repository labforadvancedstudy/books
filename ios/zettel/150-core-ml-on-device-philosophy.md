# Core ML: 프라이버시가 곧 성능

## Core Insight
On-device AI는 단순한 기술적 선택이 아니라 프라이버시와 성능을 동시에 달성하는 철학적 전환이다.

```swift
// 모델 로딩은 한 번만
guard let model = try? MLModel(contentsOf: modelURL) else { return }

// 예측은 즉시
let prediction = try model.prediction(from: input)
```

Core ML의 혁명적 측면:
1. **Zero latency**: 네트워크 없이 즉시 결과
2. **Privacy by design**: 데이터가 기기를 떠나지 않음
3. **Offline capability**: 인터넷 없이도 작동
4. **Battery efficiency**: Neural Engine이 CPU보다 효율적

가장 놀라운 것은 **모델 압축 기술**이다:
- Quantization: Float32 → Int8로 75% 크기 감소
- Pruning: 불필요한 연결 제거
- Knowledge Distillation: 큰 모델의 지식을 작은 모델로 전수

Apple의 전략은 명확하다: "서버에 의존하지 않는 AI". 이는 개인정보 보호뿐만 아니라 사용자 경험의 질적 향상을 의미한다.

현실적 한계:
- 모델 크기 제한 (보통 수십 MB)
- 복잡한 추론은 여전히 느림
- 훈련은 불가능 (추론만 가능)

하지만 트렌드는 명확하다: 더 작고, 더 빠르고, 더 효율적인 온디바이스 모델.

## Connections
→ [[151-neural-engine-architecture]]
→ [[152-create-ml-workflow]]
→ [[153-model-optimization-techniques]]
← [[019-ai-on-device-philosophy]]

---
Level: L5
Date: 2025-08-16
Tags: #ios #coreml #ai #privacy #performance #on-device