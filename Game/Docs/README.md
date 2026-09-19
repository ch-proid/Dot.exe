# Dot.exe 문서 구조

## current

현재 구현과 기획 판단에 사용하는 문서다.

1. `PROJECT_CONTEXT.md` — 프로젝트 전체 맥락, 최근 결정, 미확정 항목, 다음 재개 지점
2. `CORE_GAME_RULES.md` — 최상위 정본
3. `DEVELOPMENT_SPEC.md` — 구현 기준
4. `GAME_DESIGN.md` — 경험·콘텐츠 방향
5. `ARCHITECTURE.md` — 코드 구조와 의존 방향
6. `DEVELOPMENT_PIPELINE.html` — 단계별 개발·검증 절차
7. `LOCALIZATION_KO_REFERENCE.md` — 한국어 용어·UI·연구명 검토표

문서가 충돌하면 `CORE_GAME_RULES.md`의 우선순위를 따른다.

## history

검토 과정과 과거 결정 기록이다. 현재 규칙의 근거를 추적할 때만 사용하며 구현 기준으로 사용하지 않는다.

## tests

단계별 사람 테스트 체크리스트를 둔다.


작업을 재개할 때는 `PROJECT_CONTEXT.md`를 먼저 읽고, 실제 규칙 판단은 `CORE_GAME_RULES.md`를 따른다.
