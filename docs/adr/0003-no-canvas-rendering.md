# ADR-0003: No Canvas Rendering — HTML/CSS Only

- Status: Accepted (2026-05-06)
- Constraints affected: A1, A2, A3, B4
- Related features: F-008~F-012, F-020

## Context

C3의 핵심 사용 맥락 = _채용 통과율_. ATS(Applicant Tracking System) 텍스트 추출, SEO 인덱싱, 접근성 도구(스크린 리더) 작동이 비즈니스 가치 1순위. 사전 자료조사에선 Konva.js(canvas freeform)를 권장했지만, ATS·SEO·a11y 통과율 0이라 거부.

## Decision

Canvas 렌더링 엔진 사용 금지. `konva`, `fabric`, `paper.js`, `react-konva` 등 import 자체 차단 (`.eslintrc.cjs`의 `no-restricted-imports`). 사용자 페이지 출력은 _오직 HTML/CSS_. 그래픽 합성이 필요한 경우만 SVG로 해결.

자유 배치는 ADR-0004 (섹션 + 자유 배치 + 자동 반응형) 참조 — Canvas 없이도 자유도 확보.

## Consequences

- ATS 통과율 100%, SEO 인덱싱 가능, a11y 자동 처리
- 자유 배치 정밀도는 CSS Grid + viewport-relative로 달성 (B4)
- 일부 시각 효과(복잡한 합성, 캔버스 페인팅)는 불가
- 시각 레이어와 의미 레이어 통합 — 시맨틱 HTML 강제(A2) 자연

## Alternatives Considered

- **Konva.js (자료조사 권장)**: ATS 0, SEO 0, a11y 0 — 비즈니스 가치 정면 충돌
- **Hybrid (Canvas + DOM 텍스트 오버레이)**: 복잡도 폭발, 솔로엔 비현실
- **WebGL (three.js 등)**: 추가 학습 비용 + 같은 ATS/SEO 문제
