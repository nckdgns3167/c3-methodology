# ADR-0011: 한국어 타이포 강제 (Pretendard + 자간/행간 best practice)

- Status: Accepted (2026-05-07)
- Constraints affected: A3 (확장)
- Related features: F-008~F-012, F-020
- Related ADRs: ADR-0009 (DESIGN.md SSOT), ADR-0003 (HTML/CSS only)

## Context

C3 주 사용자 = 한국어 구직자 (정부지원사업 _모두의창업_ + 베타 출시 2026-11-30). 한국어 페이지가 영문 기본값 typography(Inter, 영문 자간/행간)로 박히면 가독성·전문성 손상 → 채용 통과율 직결.

KR 디자인 표준(토스/네이버/다음/카카오 등)은 일관:

- **폰트**: Pretendard
- **자간**: `-0.02em` (영문은 0)
- **행간**: heading `1.3~1.4` / body `1.5~1.6`

ADR-0009가 DESIGN.md를 시각/UX SSOT로 박았으나 *어떤 폰트·디테일을 박을지*는 미정. F-008 작업 시 매번 결정하면 일관성 손실. 거시 결정으로 _지금_ 박는 게 맞음.

## Decision

C3 사용자 페이지 typography는 다음 best practice 강제:

**한국어 폰트 (1차)**:

```
--font-ko: 'Pretendard Variable', 'Pretendard', -apple-system, BlinkMacSystemFont,
           system-ui, Roboto, 'Helvetica Neue', 'Segoe UI',
           'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic', sans-serif;
```

**다국어 분기 (E16 정합)**:

```
--font-en: Inter, -apple-system, BlinkMacSystemFont, system-ui, sans-serif;
--font-ja: 'Noto Sans JP', -apple-system, system-ui, sans-serif;
```

페이지 `lang` 속성 기반 자동 분기. Phase 1엔 한국어만 활성, 다국어 도입(E16) 시 자동 분기.

**자간/행간 (KR best practice)**:

- 자간 (letter-spacing) 기본값: `-0.02em` (한국어). 영문 전용 텍스트는 `0` 또는 `+0.01em`.
- 행간 (line-height) heading: `1.3~1.4`
- 행간 body: `1.5~1.6` (`1.55` 권장)
- 행간 caption: `1.4`

**DESIGN.md 강제**:

- typography 섹션에 위 best practice를 *placeholder가 아닌 실제 값*으로 박음
- F-007 build-tokens.ts가 DESIGN.md → CSS 변수 derive (`--font-ko`, `--letter-spacing-default`, `--line-height-body` 등)
- F-008 Hero 작업 시 heading/body 폰트 size·weight 추가 결정 (자간/행간은 이 ADR 따름)

## Consequences

- 채용 통과율 직결 한국어 가독성 보장 (영상 _AI 슬롭_ 회피의 핵심)
- Pretendard 의존 추가 — npm `pretendard` 자체 호스팅 권장 (CDN 의존 SEO·성능 영향 회피, F-008 결정)
- 다국어 분기 코드 약간 복잡화 (E16 정합 비용)
- _어디서나 보는 KR 서비스_ 위험 — C3만의 색깔(채용·경력 톤, accent color)은 F-008 작업 시 결정 (ADR-0012 벤치마킹 워크플로 + plan 0003 Pre-mortem 참조)

## Alternatives Considered

- **영문 기본 (Inter, system-ui)**: 한국어 부자연, 채용 통과율 우려 → 거부
- **Noto Sans KR**: Pretendard보다 ~2배 무거움, KR 디자인 표준 X → 거부 (fallback로만)
- **System 기본 (Apple SD Gothic Neo / Malgun Gothic)**: OS별 차이 큼, 일관성 부재 → fallback로만
- **자체 폰트 (브랜딩 차별화)**: Phase 1 비용 폭발, 라이선스 부담 → Phase 2 검토
- **자간 0 / 행간 1.6 단일값**: 한국어 표준과 미세 어긋남, 가독성 손실 → 거부
