# ADR-0012: 외부 디자인 시스템 reference 정책 (원티드 몽타주 + Coolicons)

- Status: Accepted (2026-05-07)
- Constraints affected: (해당 없음 — 메타 작업 정책)
- Related features: F-008~F-012
- Related ADRs: ADR-0009 (DESIGN.md SSOT), ADR-0011 (한국어 타이포)

## Context

YouTube 영상 *"AI로 만든 제품이 안 팔리는 이유"*의 _치트키_: 검증된 KR 시스템(원티드 몽타주 등)에서 컴포넌트 _구조_ 가져와 색깔만 입히기. 솔로엔 빠른 길. 또 토스/리니어 등 reference 서비스 스크린샷 → AI 클론 → DESIGN.md 추출 워크플로.

또 영상 추천: Coolicons처럼 _일관 규칙_ 아이콘 라이브러리 30개 큐레이션. 무료 아이콘 섞으면 어수선.

우리 F-008(Hero 앵커) 자체 빌드 가정. 외부 reference 활용 정책 없으면 매번 결정 + 솔로 시간 폭발 → 베타 마감(2026-11-30) 위험.

## Decision

외부 디자인 시스템을 *reference*로 활용 허용. 단 비즈니스 차별화는 자체 유지.

**활용 패턴**:

- **컴포넌트 구조 차용**: 원티드 몽타주(https://montage.wanted.kr) 등 검증된 KR 시스템에서 _레이아웃·간격·계층 구조_ 가져오기 OK.
- **아이콘 라이브러리**: Coolicons(https://coolicons.cool) 채택 — 일관 규칙 + 무료 + 상업 이용 OK. F-008 작업 시 30개 큐레이션 → `src/lib/icons/` 또는 manifest로 export.
- **벤치마킹 워크플로** (F-008 작업 _전_):
  1. 토스/리니어/원티드 몽타주 1~2 reference 선택
  2. 스크린샷 또는 URL을 AI(Claude/Gemini)로 클론 분석 또는 design.md 추출
  3. 추출된 값을 DESIGN.md 1차 박음 (typography는 ADR-0011 한국어 best practice 우선)
  4. 우리만의 색깔(채용·경력 톤, accent color, hero composition) 추가
  5. 시간 박스 1d 이하 — 초과 시 자체 진행

**라이선스 강제**:

- _상업 이용 + 수정 자유_ 라이선스만 사용 (Apache 2.0, MIT, OFL, CC-BY 4.0 등)
- 원티드 몽타주: 상업 이용 + 수정 자유 (영상 명시, 작업 시점 재검증)
- Coolicons: 무료 + 상업 이용 (작업 시점 라이선스 정독 필수)
- 라이선스 파일/조건은 F-026(법적 페이지) 작업 시 `docs/legal/` 또는 `LICENSES.md`에 명시

**우리 차별화 보존 (외부 reference 안 따르는 부분)**:

- ATS·SEO·접근성 보존 (제약 #1, #2, #17) — 외부 시스템이 보장 안 함
- 한국어 타이포 디테일 (ADR-0011) — 우리가 명시 박음
- JSON Resume 호환 데이터 (D14, ADR-0002)
- 채용·경력 강조 색깔 — F-008 작업 시 결정

## Consequences

- F-008~F-012 시간 절약 (자체 빌드 vs reference 차용)
- 일관성 ↑ (검증된 시스템 기반)
- 라이선스 책임 발생 — methodology repo에 reference 사용 명시 (투명성)
- 컴포넌트 구조의 _originality_ 일부 양보 — 우리 차별화는 데이터/제약/한국어 디테일/채용 톤에 집중

## Alternatives Considered

- **100% 자체 빌드**: 솔로 시간 폭발, 일관성 약함 → 거부
- **100% 외부 그대로**: 차별화 0, 라이선스 위험 → 거부
- **Tailwind UI / shadcn/ui 등 OSS 컴포넌트 라이브러리**: Phase 2 검토 (현재 Tailwind 3 + 우리 컴포넌트)
- **유료 디자인 시스템**: 비용 + 라이선스 제한 → Phase 2 검토
- **벤치마킹 안 하기**: AI 슬롭 위험, 영상 인사이트 무시 → 거부

## 의도적으로 안 할 것

- 외부 시스템 _그대로 복사_ (차별화 0)
- 라이선스 미확인 상태 사용
- 벤치마킹 단계가 1d 초과 (시간 박스 — 영상 치트키의 의도 위배)
- DESIGN.md 거치지 않은 직접 토큰 (ADR-0009 위배)
- 무료 아이콘 섞어 사용 (Coolicons 외 라이브러리 — 일관성 깨짐)
