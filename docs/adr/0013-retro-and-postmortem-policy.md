# ADR-0013: Retro & Postmortem 정책 (Plan-as-Code 통합)

- Status: Accepted (2026-05-07)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related ADRs: ADR-0010 (Plan-as-Code), ADR-0008 (feature_list.json as Backlog SSOT)

## Context

PDF *"회고 적용 방법 조사"*가 5 분류(Sprint Retro / Postmortem / ADR / Feature Retro / Working Log) + 4 트렌드(Docs-as-Code / Blameless / 5 Whys+Action item / Lightweight) 짚음. 우리 환경(1인+AI 위임, plan-as-code 채택) 비교:

- Sprint Retro — 1인엔 부담 vs 가치, skip 권장
- ADR — 12개 합의됨
- Working Log — `claude-progress.md` 합의됨
- Postmortem (사고) — _미정_, 사고 발생 시 어디 적을지·템플릿 부재
- Feature Retro — _미정_, PDF는 별도 파일(`docs/retros/feature/F-XXX.md`) 권장

PDF가 미언급한 우리만의 발견: **Pre-mortem ↔ Post-mortem 페어링**. ADR-0010이 plan에 Pre-mortem 강제 박았는데, post-mortem(retro)이 그 거울. 둘이 한 plan 파일에 있어야 _예측 vs 실제_ 비교가 자연 — 학습 가치의 코어.

## Decision

**1. Feature retro = plan 파일 안 통합** (PDF의 별도 파일 권장과 다름):

- 별도 `docs/retros/feature/F-XXX.md` 파일 _안 만듦_
- 대신 plan 파일에 `## Post-mortem` 섹션 강제 (Pre-mortem 짝)
- feature done 처리 시 `validate-feature-list.ts`가 Post-mortem 비어있지 않은지 검증
- 형식 (5~10줄): 추정 vs 실제 / 잘된 것 / 어려웠던 것 / 다음 feature에 가져갈 것 / 백로그 영향
- DRY + Pre-mortem ↔ Post-mortem 한 파일 비교

**2. Postmortem (사고) = 별도 디렉토리** (PDF 동의):

- `docs/postmortems/YYYY-MM-DD-<title>.md`
- 사건 발생 시 `_template.md` 복사 → 채움
- 1페이지 이내 (Severity / Detected / Resolved / Duration / Affected / What happened / Timeline / 5 Whys / Action items / What worked / What didn't)
- 사소한 디버깅은 `claude-progress.md`에 한두 줄 — 경계: "다른 사람(또는 6개월 뒤의 나)이 같은 문제 겪을 가능성 있는가?" yes면 postmortem

**3. Sprint retro = 안 함**:

- 1인 + AI 환경에 부담 > 가치 (PDF 권장 일관)
- 합류자 발생 또는 Phase 2에서 재검토

**4. Action item → 백로그 통합** (PDF 트렌드 #3):

- postmortem 또는 plan Post-mortem의 `- [ ]` 항목은 _반드시_ feature_list.json에 매핑되는 새 F-XXX (또는 기존 feature 보강)
- "회고 문서만 남고 실행 안 됨" 가장 흔한 실패 차단

**5. Lightweight templates** (PDF 트렌드 #4):

- Postmortem 1페이지 이내
- Plan Post-mortem 5~10줄
- 7~9 항목 무거운 양식 X

**6. Blameless 원칙** (PDF 트렌드 #2):

- _사람이 아니라 시스템_ 분석
- "X가 실수했다" X / "X 실수를 막을 가드레일이 없었다" O
- 1인이라도 자기에게 적용 — "내가 멍청해서" X / "스키마 검증이 없어서" O
- Action item이 *사람의 더 잘하기*가 아닌 *시스템 변경*으로 나와야

**7. validate-feature-list.ts 보강 (cross-validation 6번째)**:

- feature status `done` 시 plan 파일 (`docs/plans/NNNN-...md`) 매핑 + `## Post-mortem` 섹션 + 비어있지 않음 검증
- placeholder 또는 빈 섹션은 거부
- plan 파일 매핑 못 찾으면 warn (강제 reject 아님 — plan-as-code 적용 전 features 회귀 가능)

## Consequences

- 회고가 *기계적 게이트*에 묶임 (PDF 전략 A) — 안 쓰면 done 안 닫힘. 의지력 의존 X
- Pre-mortem ↔ Post-mortem 학습 페어링 자연 형성 (한 plan 파일에 비교 가능)
- 사고 회고는 별도 흐름 (날짜 기반), feature 회고는 plan 통합 (id 기반) — SSOT 분산 회피
- Sprint retro 미실행으로 1인 부담 0
- Action item 백로그 통합으로 _문서 → 실행_ 직접 연결

## Alternatives Considered

- **PDF 권장: docs/retros/feature/F-XXX.md 별도 파일** — plan-as-code(ADR-0010) 환경에서 SSOT 분산. Pre-mortem ↔ Post-mortem 비교 기능 약함 → 거부
- **Sprint retro 도입** — 1인+AI ROI 낮음 (PDF도 동의) → Phase 2 재검토
- **회고 안 함** — 솔로 환경 가장 흔한 길. 학습 손실, 같은 실수 반복 → 거부
- **Notion/Linear retro** — SSOT 분산 (ADR-0008과 동일 사유) → 거부
- **/postmortem 슬래시 명령** — Phase 2 검토 (지금은 수동 템플릿 + 자동 검증)

## 의도적으로 안 할 것

- Sprint retro 디렉토리 (`docs/retros/sprint/`) — Phase 2까지 비활성
- Feature retro 별도 파일 (`docs/retros/feature/`) — plan 통합으로 대체
- 회고 형식 7~9 항목 무거운 양식 — Lightweight 강제
- 모든 사소한 디버그를 정식 postmortem화 — 사소한 건 claude-progress.md
- Action item을 plan Post-mortem 안에만 두기 — feature_list.json 매핑 강제
