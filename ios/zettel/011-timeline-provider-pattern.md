# TimelineProvider: 시간의 설계

## Core Insight
TimelineProvider는 미래를 계산하는 예언자다. Widget이 언제 무엇을 보여줄지 미리 결정한다.

placeholder()는 데이터가 없을 때의 모습이다. 스켈레톤 UI, 로딩 상태가 아니다. "이런 종류의 정보가 여기 표시됩니다"라는 약속이다. 텍스트는 redacted되고 이미지는 흐릿하다.

getSnapshot()은 즉각적인 프리뷰다. Widget Gallery에서, 추가할 때 보이는 모습이다. 실제 데이터를 가져올 시간이 없다. 그럴듯한 가짜 데이터로 Widget의 가치를 보여준다.

getTimeline()이 진짜다. Entry 배열과 다음 업데이트 정책을 반환한다. 각 Entry는 Date와 함께 미래의 특정 시점을 표현한다. 시스템은 이 타임라인을 따라 Widget을 업데이트한다.

.atEnd, .after(date), .never - 리로드 정책이 배터리 수명을 결정한다. 너무 자주 업데이트하면 배터리를 잡아먹고, 너무 드물면 정보가 낡는다. 균형의 예술이다.

## Connections
→ [[038-widget-configuration]]
→ [[039-widget-bundle-strategy]]
← [[010-widget-as-glimpse]]

---
Level: L2
Date: 2025-08-15
Tags: #ios #widget #timeline #provider