# Luna Chat Coder entry point

When repository development is requested from a chat surface with a disposable sandboxed code-execution environment, read `.agents/skills/luna-chat-coder/SKILL.md` before working on the repository task.

Loading the skill is a readiness step, not a reason to use GitHub Actions. Normal engineering work should stay in the chat sandbox work container when it is available and sufficient.

The repository itself defines its runtimes, services, dependencies, architecture, build system, and verification requirements. Luna Chat Coder supplies continuity, exact transport, and bounded fallback capability; it does not introduce a development methodology or substitute technologies merely because they are easier to run.

Treat exact GitHub commit and PR state as durable source truth, preserve unrelated work, and do not make access to the user's computer a dependency of the workflow.

When this repository is used as a template, keep this entry point and add the project's own engineering instructions alongside it.

---

# 개발 규칙

코드 작업 전에 `ARCHITECTURE.md`를 먼저 읽는다. 게임 규칙은 `CORE_GAME_RULES.md`가 원본이고, 구현 기준은 `Docs/Dot.exe_개발명세서.md`다. 문서끼리 어긋나면 `CORE_GAME_RULES.md` 1절의 순서를 따른다.

## 코드

- 기존 public interface, 이벤트 이름, ID, 저장 형식을 임의로 바꾸지 않는다.
- 요청과 관계없는 파일을 수정하지 않는다. 기능 하나를 넣으면서 대규모 리팩터링을 하지 않는다.
- 새 기능을 만들기 전에 기존 확장 지점을 찾는다.
- UI에서 게임 상태를 직접 수정하지 않는다. Command나 Transaction을 서비스에 보낸다.
- Renderer에 게임 규칙을 넣지 않는다.
- `src/features/`의 domain과 system은 DOM, `presentation/`, `input/`, `Localizer`를 import하지 않는다.
- balance 값을 코드에 하드코딩하지 않는다. `src/data/balance/`에 둔다.
- 화면에 나오는 글을 코드나 데이터에 직접 적지 않는다. `Locale/en.json`에 키를 먼저 넣고 `ko.json`을 맞춘다.
- `any`로 타입 오류를 덮지 않는다.
- 게임 결과에 영향을 주는 난수는 `gameplayRandom`만 쓴다. `Math.random()`을 쓰지 않는다.
- SaveData를 바꾸면 버전을 올리고 migration과 테스트를 추가한다.
- feature 사이의 직접 의존보다 service contract나 event를 먼저 검토한다.

## 규칙을 바꿔야 할 때

구현하다가 규칙이 비어 있거나 서로 어긋나는 것을 발견하면 코드에서 임의로 정하지 않는다. 멈추고 사람에게 묻는다. 정해지면 `CORE_GAME_RULES.md`를 먼저 고치고 변경 기록에 적은 뒤 코드를 맞춘다.

## 완료 기준

- `npm run check`가 통과한다. (타입 검사, 테스트, 현지화 키 검사)
- 새 규칙에는 테스트가 있다.
- 작업 보고에 변경한 파일, 기존 기능에 주는 영향, 테스트 결과를 적는다. 실패한 테스트는 실패했다고 적는다.

## 단계 진행

`Docs/Dot.exe_개발_파이프라인.html`의 단계를 하나씩 밟는다. 한 단계가 끝나면 에이전트 배속 테스트, 브라우저 배속 테스트, 사람 테스트를 순서대로 거친다(`CORE_GAME_RULES.md` 30.2절). 사람이 승인하기 전에는 다음 단계의 코드를 쓰지 않는다. 사람 테스트용 체크리스트는 `Docs/테스트/`에 둔다.

