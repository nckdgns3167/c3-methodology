# ADR-0007: WIP = 1

- Status: Accepted (2026-05-06)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related features: F-005 (validate-feature-list가 강제)

## Context

솔로 + AI 위임 환경에서 컨텍스트 스위칭 비용이 매우 큼. 미완성 작업 누적 시 추적 실패 → "지난번에 뭐 하다 멈췄지?" 회복 비용 폭발. 팀 환경의 WIP 3-5는 솔로엔 과.

## Decision

`feature_list.json`의 `wip_policy.max_in_progress: 1`을 *절대값*으로 강제.

- 새 작업 시작 시 status=in*progress인 feature가 *없어야\_ 함
- 차단 발생 시 paused로 전환 + `reason` 기록 강제 (`active_history`에 paused 이벤트 + reason)
- 다른 작업으로 옮긴 뒤 영원히 안 돌아오는 패턴을 하네스가 막음
- `wip_lock=false` (의존성 업데이트, 보안 패치 등 메타 작업)는 카운트 제외

`scripts/validate-feature-list.ts`가 이 정책 자동 강제 (cross-validation 1번).

## Consequences

- 한 번에 한 작업의 _완결성_ 보장
- 컨텍스트 보존 (paused 이유가 active_history에 영구)
- 일정 추정 정확도 향상 (병렬 작업 가정 제거)
- 일부 _작업 사이 대기 시간_(blocker) 동안은 진짜 idle — 받아들임

## Alternatives Considered

- **WIP 무제한**: 솔로엔 적, 미완성 누적 → "어디까지 했지" 비용 폭발
- **WIP 3-5 (팀 표준)**: 솔로엔 과, 컨텍스트 스위칭 비용 더 큼
- **WIP 1 + paused 무제한**: 우리 결정과 동일 (paused는 카운트 제외)
