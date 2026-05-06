# c3-methodology

C3 (Create your Career on Canvas) 프로젝트의 *방법론* 공개 저장소.

코드는 비공개 (`c3` private repo). 이 저장소엔 **어떻게 일하는지**만 공개:

- **[AGENTS.md](./AGENTS.md)** — 프로젝트 헌법. 17개 비타협 제약 + 운영 원칙 + Section 0 SSOT 표
- **[docs/adr/](./docs/adr/)** — Architecture Decision Records (10개). 비타협 결정의 사유 영구화
- **[docs/glossary.md](./docs/glossary.md)** — 용어 사전 (35개, 8 카테고리). 방법론 학습 자료
- **[DESIGN.md](./DESIGN.md)** — 시각/UX SSOT (Google Labs 사양, Apache 2.0). 현재 스캐폴드 단계

## C3 프로젝트

**리크루팅 특화 노코드 캔버스 플랫폼**. ATS·SEO·a11y 통과율을 보존하는 *HTML/CSS-only* 페이지 빌더. 1인 개발자 + AI 에이전트 위임 체제로 **베타 출시 2026-11-30** 목표 (정부지원사업 *모두의창업*).

## 왜 방법론을 공개?

1인 + AI 위임 환경에서 *Harness Engineering*을 어떻게 박는지의 실험 + 학습 자료. ADR / glossary / SSOT 분리 / WIP=1 / Plan-as-Code(ADR-0010) 패턴은 다른 솔로·소규모 프로젝트에도 적용 가능 — 자유롭게 참고.

## 핵심 패턴

- **SSOT 분리** (AGENTS.md Section 0): 헌법 / 백로그 / 시각의도 / 결정근거 / 실행계획 / 용어 / done이벤트의 SSOT를 *하나씩* 분리. CLAUDE.md / .cursorrules 등 도구별 메모리 파일은 모두 stub.
- **WIP=1 엄격** (ADR-0007): 동시 진행 작업 정확히 1개. paused 시 reason 강제. 솔로엔 컨텍스트 스위칭 비용 최소화.
- **Pass-state Gating** (AGENTS.md 3.3): husky pre-commit (1차) + GitHub Actions (2차). init.sh 한 단계 실패 시 비제로 종료.
- **Plan-as-Code** (ADR-0010): 외부 도구(Notion/Linear) 회귀 → markdown + git. frontmatter + Pre-mortem 강제 + supersede 체인 진화 추적.
- **결정 타이밍**: 거시 결정(스택·17제약·페이즈)은 미리 박음. 미시 결정(필드 길이·UI 임계값)은 작업 시점 컨텍스트 보고.
- **도구 중립**: AGENTS.md가 SSOT, 어느 AI 코딩 에이전트(Claude Code / Codex / Cursor / Windsurf 등)에도 동등 적용.

## 코드 저장소

코드는 https://github.com/nckdgns3167/c3 (private). 베타 직전 public 전환 여부 재결정.

## sync 정책

이 저장소는 c3 코드 저장소에서 *수동 큐레이션* 동기화. 분기별 또는 베타 직전 sync. 자동 mirroring은 Phase 2 검토.

## 라이선스

현재 라이선스 미명시. 학습·참고 목적 자유 사용 권장. 출처(`https://github.com/nckdgns3167/c3-methodology`) 표기 부탁. CC-BY 4.0 정식 명시는 추후 검토.
