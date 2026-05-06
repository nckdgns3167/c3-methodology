# ADR-0006: AGENTS.md as SSOT (CLAUDE.md is Stub)

- Status: Accepted (2026-05-06)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related features: F-001 (CLAUDE.md stub 산출물)

## Context

멀티 AI 코딩 에이전트(Claude Code, Codex, Cursor, Copilot, Windsurf 등) 환경에서 도구별 메모리 파일 다름:

- Claude Code: `CLAUDE.md`
- Cursor: `.cursorrules`
- Copilot: `.github/copilot-instructions.md`
- 이외 도구별 파일

각 도구별로 룰을 다시 적으면 동기화 비용 폭발 + 불일치 위험. 도구 락인 위험.

## Decision

`AGENTS.md`를 운영 규칙·17 비타협 제약·작업 절차의 SSOT로 박는다. 도구별 메모리 파일은 모두 AGENTS.md를 가리키는 *stub*만 두고 규칙 중복 정의 _금지_.

stub 형식: "This repo uses AGENTS.md as SSOT. Read @AGENTS.md, @feature_list.json, @claude-progress.md."

## Consequences

- 룰 변경은 AGENTS.md 한 군데만 → 동기화 비용 0
- 새 AI 도구 도입 시 stub 한 줄만 추가 → 도구 중립
- AGENTS.md 변경 PR이 모든 에이전트에 즉시 반영
- 도구별 _특수 행동_(예: Claude Code 슬래시 커맨드)은 별도 영역으로 분리 (지금은 없음)

## Alternatives Considered

- **각 도구별 풀 룰 복제**: DRY 위반, 불일치 위험
- **도구별 부분 specialization**: 어디까지 공유/특화인지 모호함, 결정 비용 폭발
- **CLAUDE.md only (Claude 락인)**: 크로스 AI 전략과 정면 충돌 (Codex 등 사용 가능성 차단)
