# Dot.exe 개발 규칙

이 파일은 `Game/` 아래의 게임 작업에만 적용한다. Luna Chat Coder 자체의 동작 규칙은 저장소 루트 `.agents/`에 있으며 서로 섞지 않는다.

코드 작업 전에 `Game/Docs/current/ARCHITECTURE.md`를 읽는다. 게임 규칙의 정본은 `Game/Docs/current/CORE_GAME_RULES.md`, 구현 기준은 `Game/Docs/current/DEVELOPMENT_SPEC.md`, 경험과 서사 방향은 `Game/Docs/current/GAME_DESIGN.md`다.

## 코드

- 기존 public interface, 이벤트 이름, ID, 저장 형식을 임의로 바꾸지 않는다.
- 요청과 관계없는 파일을 수정하지 않는다. 기능 하나를 넣으면서 대규모 리팩터링을 하지 않는다.
- 새 기능을 만들기 전에 기존 확장 지점을 찾는다.
- UI에서 게임 상태를 직접 수정하지 않는다. Command나 Transaction을 서비스에 보낸다.
- Renderer에 게임 규칙을 넣지 않는다.
- `Game/src/features/`의 domain과 system은 DOM, presentation, input, Localizer를 import하지 않는다.
- balance 값은 `Game/src/data/balance/`에 둔다.
- 화면 글은 localization key를 사용하고 `Game/Locale/en.json`, `ko.json`에서 관리한다.
- `any`로 타입 오류를 덮지 않는다.
- 게임 결과에 영향을 주는 난수는 gameplay RNG만 쓴다.
- SaveData를 바꾸면 버전을 올리고 migration과 테스트를 추가한다.
- feature 사이 직접 의존보다 service contract나 event를 먼저 검토한다.
- 기준 화면은 1080×1920 세로, 논리 화면은 216×384다. 화면비 적응은 ResponsiveShell에서 처리하며 simulation 좌표를 기기별로 바꾸지 않는다.

## 규칙을 바꿔야 할 때

구현 중 규칙이 비어 있거나 충돌하면 코드에서 임의로 정하지 않는다. `CORE_GAME_RULES.md`를 먼저 수정하고 변경 기록을 남긴 뒤 나머지 문서와 코드를 맞춘다.

## 완료 기준

- `npm run check` 통과
- 새 규칙에 테스트 존재
- 관련 없는 파일 수정 없음
- SaveData / event contract 영향 확인
- 변경 파일, 기존 기능 영향, 실제 실행한 테스트를 보고

## 단계 진행

`Game/Docs/current/DEVELOPMENT_PIPELINE.html`의 단계를 하나씩 밟는다. 에이전트 배속 → 브라우저 배속 → 사람 테스트를 거치며, 사람 테스트 체크리스트는 `Game/Docs/tests/`에 둔다.
