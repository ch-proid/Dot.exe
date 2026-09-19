# Dot.exe 개발 규칙

이 파일은 `Game/` 아래의 게임 작업에만 적용한다. Luna Chat Coder 자체의 동작 규칙은 저장소 루트 `.agents/`에 있으며 서로 섞지 않는다.

Dot.exe 관련 작업을 시작할 때는 먼저 `Game/Docs/current/PROJECT_CONTEXT.md`를 읽어 현재 프로젝트 맥락과 최근 재개 지점을 파악한다. 특히 `작업 중 발생한 실수와 교훈` 절을 함께 확인해 이미 발생한 작업 오류를 반복하지 않는다. 그 다음 코드 작업 전에 `Game/Docs/current/ARCHITECTURE.md`를 읽는다. 게임 규칙의 정본은 `Game/Docs/current/CORE_GAME_RULES.md`, 구현 기준은 `Game/Docs/current/DEVELOPMENT_SPEC.md`, 경험과 서사 방향은 `Game/Docs/current/GAME_DESIGN.md`다. `PROJECT_CONTEXT.md`는 이 문서들을 대체하지 않고, 결정의 이유·현재 진행 상태·미확정 항목을 이어 주는 지속 맥락 문서다.

## 간단한 작업의 병렬 처리

- 지침을 그대로 따르면 되고 별도 추론이 필요 없는 문서 작업, 간단한 코딩 작업, 서로 겹치지 않는 간단한 작업은 Luna High(`gpt-5.6-luna`, `high`) 에이전트 여러 개에 나눠 동시에 진행한다.
- 에이전트마다 맡을 범위와 완료 기준을 분명히 정하고, 같은 파일을 동시에 수정하지 않게 한다.
- 담당 에이전트는 결과를 모아 변경 사항과 필요한 검사를 확인한다. 작업끼리 의존하거나 설계·규칙 판단이 필요하면 병렬로 나누지 않는다.

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


## 지속 맥락 관리

프로젝트의 방향, 시스템 의미, UI 최종 선택, 스토리 구조, 개발 단계처럼 이후 판단을 바꿀 수 있는 중요한 결정이 생기면 `Game/Docs/current/PROJECT_CONTEXT.md`를 함께 갱신한다. 단순 오탈자나 작은 밸런스 조정은 기록하지 않는다.


## 도구 가용성 판단

GitHub 작업 가능 여부를 sandbox의 `git`/`gh` CLI 유무만으로 판단하지 않는다. GitHub Plugin/Connector는 별도 경로이므로, 직접 조작 가능 여부가 불분명하면 connector의 profile/branch/file read를 먼저 실행해 확인한다. 작업 완료는 branch/PR의 exact SHA 또는 실제 파일 재조회로 검증한 뒤 보고한다.
