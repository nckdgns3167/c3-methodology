# ADR-0002: JSON Resume as Data Contract

- Status: Accepted (2026-05-06)
- Constraints affected: D14
- Related features: F-022

## Context

사용자 데이터의 *주권*과 _이식성_ 보장 필요. C3 사용자가 자기 데이터를 다른 도구로 가져갈 수 있어야 신뢰 확보. 자체 schema는 호환성 0, 산업 표준이 있으면 활용 우선.

## Decision

JSON Resume v1 표준(jsonresume.org/schema)을 데이터 계약으로 채택. 사용자 페이지의 모든 데이터는 12 표준 필드(basics, work, education, skills, projects, awards, certificates, languages, interests, references, volunteer, publications)에 매핑되거나 `meta.custom`에 보존.

`GET /api/resume/[userId].json` 엔드포인트 항상 200 + 유효 schema. 표준 필드 임의 변형/누락 금지. CI contract test로 외부 schema validator 통과 강제 (F-022).

## Consequences

- 사용자 데이터 이식성 보장 (다른 JSON Resume 호환 도구로 이동 가능)
- 스키마 변경은 표준 진화 따라가야 — 자유도 일부 포기
- 사용자 정의 필드는 `meta.custom`으로 보존 — 표준 외 데이터 손실 X
- 외부 표준이라 우리가 schema 수정 못 함 (장점이자 제약)

## Alternatives Considered

- **자체 schema**: 호환성 0, 사용자 락인
- **Schema.org Person**: UI 친화성 부족
- **HRJSON**: 덜 보편적, ecosystem 작음
