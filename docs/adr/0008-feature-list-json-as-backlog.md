# ADR-0008: feature_list.json as Backlog SSOT

- Status: Accepted (2026-05-06)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related features: F-005 (validate-feature-list)

## Context

백로그를 어디에 둘지 — GitHub Issues, Linear, Notion, 자체 JSON 파일 등. 솔로 + AI 위임에선 _AI 에이전트가 정확히 읽고 따를 수 있어야_. 외부 시스템 의존하면 API 호출 + 인증 + 동기화 부담.

## Decision

`feature_list.json`을 백로그 SSOT로 채택.

- **기계 판독 가능한 schema**: `feature_list.schema.json`(JSON Schema 2020-12)
- **작업 상태**: status (planned/in_progress/paused/blocked/done/future) + active_history (미해결 이벤트 누적, done은 git log SSOT)
- **WIP 정책**: `wip_policy.max_in_progress: 1` (ADR-0007)
- **17 제약 매핑**: `applies_constraints`로 feature별 위반 위험 큰 제약 명시
- **ADR 매핑**: `relevant_adrs`로 작업 시 먼저 읽어야 할 ADR 명시
- **인수 기준**: human(체크리스트) + verifiable(tests / type_check / lint / build_pass / migration_dry_run / files_exist / lighthouse_seo_min / schema_valid)
- **artifacts**: `files_created` 명시 — done 시 파일 존재 검증

git 추적으로 변경 이력 영구. `scripts/validate-feature-list.ts`가 cross-validation 5종 자동.

## Consequences

- 에이전트가 직접 읽고 작업 후보 선정 가능 (외부 API 호출 0)
- 외부 시스템 의존 0 (LinkedIn 양도 시 인수 비용 낮음)
- active_history는 미해결 상태만 — done 이벤트는 *git log*가 SSOT (단일 진실)
- GitHub Issues는 사용 X (Phase 1) — Phase 2 베타 후 외부 사용자 버그 보고 용도로만 활성화

## Alternatives Considered

- **GitHub Issues**: 우리 SSOT 분산 위반, AI 읽기 위해 API 호출 + auth
- **Linear**: 외부 의존 + 개인정보 우려 + 비용
- **Markdown 백로그**: 기계 판독 어려움 (정규식 파싱 부서지기 쉬움)
- **자체 DB 테이블**: 부트스트랩 비용 폭발
