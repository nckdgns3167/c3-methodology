# ADR-0010: Plan-as-Code (docs/plans/) for Plan Versioning

- Status: Accepted (2026-05-06)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related: ADR-0006 (AGENTS.md SSOT), ADR-0008 (feature_list.json as Backlog SSOT)

## Context

ADR이 *결정 근거*를 영구화하지만 _plan_(어떻게 할 것인가)은 별도. 외부 도구(Notion, Linear)는 SSOT 분산 + 도구 락인 + 검색·형상관리 약함. Claude Code의 plan 모드가 자동 생성하는 plan(`.claude/plans/<random>.md`)은 user-specific + .gitignore — 사라지거나 PC 바뀌면 추적 불가. 회고 시 "어떤 plan이 어떻게 진화했나" 추적 불가능 = 솔로+AI 환경에서 큰 학습 손실.

## Decision

`docs/plans/` 디렉토리에 plan을 *Markdown + git*으로 영구화. ADR과 동급 첫 클래스 산출물.

- **명명**: `NNNN-<topic>.md` (4자리 시퀀스, ADR 형식 일관)
- **표준 frontmatter**: id / title / status (proposed | approved | executed | superseded) / created / related_features (optional) / related_adrs (optional) / supersedes (optional) / superseded_by (optional)
- **표준 섹션**: Context / Goal / Approach / Risks / **Pre-mortem** / Verification / Status log
- **진화 추적**: plan 변경 시 _새 파일_ + 이전 plan에 `superseded_by`, 새 plan에 `supersedes` 박음. git history는 plan 내 작은 수정, supersede 체인은 큰 방향 전환.
- **Pre-mortem 강제**: 모든 plan에 "이 plan이 6개월 후 실패한다면 _왜_ 실패했을까?" 섹션 의무. 위험 발굴 + 회고 자료.

Claude Code plan 모드의 `.claude/plans/<random>.md`는 _draft 단계_. 사용자 승인 후 `docs/plans/NNNN-...md`로 백포팅.

## Consequences

- plan SSOT가 git → 외부 도구 의존 0
- 회고 시 supersede 체인으로 진화 가시화 가능
- 새 PC에서도 즉시 추적 가능 (clone만 하면 됨)
- Pre-mortem 강제로 위험 의식 향상
- ADR과 결이 다른 *실행 계획*의 자리 명확
- 각 plan의 Status log로 "이 plan은 어디까지 갔는지" 추적

## Alternatives Considered

- **Notion / Linear**: SSOT 분산, 도구 락인, 검색 약함 (ADR-0008과 동일 사유로 거부)
- **plan을 git에만 (디렉토리·명명 정책 없음)**: 어디 두고 어떻게 명명할지 매번 결정 비용
- **ADR에 plan 통합**: ADR=결정 근거 / plan=실행 계획. 결이 다름. 분리가 맞음.
- **GitHub Issues / PRs로 추적**: ADR-0008의 외부 SSOT 분산 거부와 동일 사유.
