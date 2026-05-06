# ADR-0004: Section-based Layout (Not Free-form)

- Status: Accepted (2026-05-06)
- Constraints affected: B4, B5
- Related features: F-008~F-013

## Context

사용자 욕구: (a) 자유로운 시각 표현, (b) 모바일 자동 반응형. 두 요구가 직접 충돌 — 픽셀 절대 배치 자유는 모바일 별도 레이아웃 비용을 부른다 (Framer 스타일).

솔로 + AI에서 모바일 별도 레이아웃 비용은 일정 폭발 트리거.

## Decision

(C 안) **섹션 단위 + 섹션 내 자유 배치 + 섹션별 mobileTransform 메타로 자동 반응형**.

- 5개 기본 섹션: Hero / About / Skills / Projects / Contact
- 사용자는 _섹션 내부_ 요소를 자유 배치 (CSS Grid + viewport-relative 좌표)
- 섹션 자체는 위→아래 순차 (reorder는 가능하지만 겹침 X)
- `mobileTransform: 'stack' | 'scroll' | 'hide' | 'reorder'` 메타로 모바일 동작 자동 결정 (B5)

## Consequences

- 자동 반응형 비용 거의 0 (mobileTransform 메타가 처리)
- 자유도는 섹션 내부로 제한 — 무한 자유 X (사용자에게 명시)
- F-013 캔버스 + @dnd-kit 구현이 이 모델 따름
- 섹션 추가/제거는 가능, 섹션 순서 변경 가능, 섹션 외 요소 배치 X

## Alternatives Considered

- **풀 자유 배치 (Framer 스타일)**: 모바일 별도 레이아웃 비용 폭발, 솔로 감당 X
- **100% 템플릿 (자유도 0)**: 차별화 없음, 사용자 가치 부족
- **컴포넌트 자유 + 섹션 자유**: 복잡도 폭발 + UX 학습 비용 큼
