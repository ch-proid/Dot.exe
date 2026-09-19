# Dot.exe ARCHITECTURE

> 버전: 0.2  
> 근거 문서: `Game/Docs/current/DEVELOPMENT_SPEC.md` v0.3, `Game/Docs/current/CORE_GAME_RULES.md` v1.4
> 게임 규칙은 `Game/Docs/current/CORE_GAME_RULES.md`가 정본이고, 세부 구현 기준은 명세서가 원본이다. 이 문서는 코드 작업 전 먼저 읽는 길잡이이며 새 규칙을 만들지 않는다.

---

## 1. 한눈에 보기

| 계층 | 하는 일 |
|---|---|
| `core` | 이벤트 버스, 고정 스텝 시계, Pause, 난수, 공간/수학 유틸. 게임 로직 전체가 쓰는 하부 도구. |
| `features` | 도메인별 게임 규칙과 시스템(culture, research, protocols, threats, resources, mail, narrative, save). 실제 판정과 상태 변경이 일어나는 곳. |
| `data` | 연구·프로토콜·위협·메일·서사·밸런스의 수치와 조건. 로직 없이 값과 참조만 담는다. |
| `presentation` | 렌더링과 UI. 읽기 전용 ViewModel을 읽어 그리기만 한다. |
| `input` | 사용자 입력을 Command/Transaction으로 바꿔 알맞은 서비스로 전달한다. |
| `app` | AppContext 조립과 부트스트랩. 계층을 엮기만 하고 로직을 갖지 않는다. |

디렉터리 구조(명세서 #3 축약, 최상위 2단계까지):

```
src/
├─ app/            GameApp, bootstrap, AppContext
├─ core/           events, time, random, math
├─ features/       culture, research, protocols, threats, resources, mail, narrative, save
├─ data/           research, protocols, threats, mail, narrative, balance
├─ presentation/   render, ui, audio, i18n(Localizer)
├─ input/          InputRouter, actions
└─ tests/          integration, fixtures
```

단위 테스트는 대상 파일 옆에 `*.test.ts`로 둔다. 여러 feature를 가로지르는 통합 테스트와 공용 fixture만 `src/tests/`에 둔다. 번역 데이터는 `src/` 밖의 `Game/Locale/`에 있다.

자세한 하위 구조와 각 파일 역할은 명세서 #3.

---

## 2. 의존 방향

| From | 볼 수 있는 것 | 모르는 것 |
|---|---|---|
| `features`의 domain/system | `core`, 자기 feature 내부, 다른 feature가 내보내는 service/event | `presentation`, `input`, DOM, `Localizer` |
| `features`의 Service(공개 API) | 위와 동일 | `presentation`, `input` |
| `presentation` | 각 feature가 내보내는 읽기 전용 ViewModel, `Localizer` | feature의 domain/system 내부 객체를 직접 수정하는 것 |
| `input` | `core`, feature의 Service(Command/Transaction 전달용) | 상태를 직접 바꾸는 것 |
| `data` | 없음(선언만) | features 로직을 참조하지 않는다. features가 data를 읽는 방향만 있다 |
| `app` | 전 계층 | 로직 구현 자체 |

핵심 원칙:

- UI는 상태를 직접 바꾸지 않는다. Command/Transaction을 서비스에 보낸다(명세서 #2.2, #32).
- Renderer는 규칙을 갖지 않는다. 사망·감염·에너지 생성 조건을 Renderer에 넣지 않는다(#2.3, #31).
- feature 사이는 직접 참조보다 service contract 또는 EventBus를 우선한다(#2.5, #30).
- 플랫폼에 따라 바뀌는 것은 `presentation/`, `input/`, 저장 매체뿐이다. 게임 규칙과 `features/`는 플랫폼을 모른다(정본 30).

---

## 3. 시스템 책임표

Cell 상태는 `LifeMode`(Active/Dividing/Dormant/Dead), `Health`(0~1), `Infection` 세 축으로 나뉜다(#8, 정본 11). 아래 표는 명세서에 등장하는 서비스/시스템만 담는다.

| 이름 | 하는 일 | 하지 않는 일 |
|---|---|---|
| CultureService | culture feature의 공개 진입점. 세포/필드 조회와 명령 전달 | 렌더링, UI 상태 보관 |
| CellMovementSystem | 이동만 담당(steering 합산, 속도 clamp, 경계 처리) | 에너지 소비 판정, 감염 판정, 사망 처리, 연구 해금, 렌더링(#12) |
| MetabolismSystem | 세포·지역 영양·활성 Trait Loadout으로 에너지 변화/영양 소비 계산, 생존 우선 규칙과 굶주림에 따른 Health 감소 처리 | 분열·사망 확정(#13) |
| DivisionSystem | 분열 조건 판정, 개체 수 상한 검사, 분열 비용 예약, 완료 시 에너지를 부모·자식에 분배 | 사망 확정(#14) |
| DeathSystem | PendingDeath 표시와 사망 확정을 한 곳에서 순서대로 처리, 사망 원인 기록, `cell:died` 발행 | Narrative/통계를 직접 갱신(이벤트 구독으로 위임)(#15) |
| ResearchService | 연구 해금 조건·비용 검증, `PROTOCOL_RUNNING` 등 거부 사유 반환, 프로토콜 보상을 통한 연구 지급 처리 | 세포 행동 구현(#19) |
| TraitLoadoutService | 전역 Trait Loadout 변경 검증(Hard Limit 초과만 거부), Running 중 변경 거부 | Safe Limit 초과 거부(패널티만 적용)(#20) |
| ResourceService | `energy`/`data`/`nutrientReserve` 값을 `trySpend`/`add`로만 변경, reason 기록 | 외부의 직접 값 수정 허용(#21) |
| ProtocolService / ProtocolRunner | Idle→Preparing→Running→Completed 상태 전환, phase/action 실행, objective 판정, 수동·자동 시작 모두 `transitionToRunning` 사용 | 체크포인트 저장 형식 자체 관리(#22, #23) |
| CheckpointService | Preparing 진입 시 정확히 한 번 체크포인트 생성(SaveData와 같은 형식), 실패 시 전체 복원 | 일반 저장(SaveService)과 슬롯 공유(#22.2) |
| MailService | 수신 메일 관리, 읽음 상태, 신규 메일 이벤트, 정렬 | 스토리 조건 판정(#25) |
| NarrativeService | 조건 확인, 스토리 플래그 갱신, 메일 큐 요청, 연출 이벤트 요청 | 게임 상태 직접 조작(#26) |
| ObservationService | 실제 행동을 관찰해 `ObservationRecord` 생성, `eventTick`에 실제 simulation tick 기록 | 낮은 검사 주파수를 이유로 발생 시각을 관찰 시각으로 대체(#25.1) |
| DiscoveryService | 발견을 종류별 최초 1회만 기록하고 즉시 DATA 지급 | 미확정(Pending) DATA 보관(#21.1) |
| EmergenceService | 스토리 트리거용 Evidence/Capability 계산 | 세포 능력 자체 구현(#28) |
| SaveService | `SaveDataV1` 명시적 직렬화, 일관된 상태에서만 저장, 버전별 migration 적용 | 런타임 객체 전체 직렬화(#33, #35) |
| PauseController | `PauseSource` 집합으로 Pause 상태 관리(add/remove/isPaused) | 단일 boolean으로 단순화(#5.1) |
| InputRouter | 입력을 `SimulationCommand`/`Transaction`으로 변환해 서비스로 전달 | 상태 직접 변경(#32) |
| Renderer(CultureRenderer 등) | 읽기 전용 ViewModel(`CultureRenderState`)을 그리기만 함 | 사망·감염·에너지 생성 등 규칙 포함(#2.3, #31) |

---

## 4. Simulation tick 순서

한 tick의 순서는 고정이며 임의로 바꾸지 않는다(정본 10.1).

```
1. Simulation Command 적용
2. Protocol 예약 변화 적용
3. Environment / Field 갱신
4. Spatial Index 생성
5. Cell Sensing
6. Steering 계산
7. Movement
8. Boundary 처리
9. Spatial Index 재구축

10. 영양/공격/감염 요청 계산
11. 요청을 동시 해석
12. 피해·영양·감염 결과 적용

13. Metabolism
14. Stress / Recovery
15. PendingDeath 판정

16. Division 진행도 갱신과 조건 판정
    - PendingDeath 개체 제외

17. Death 확정
18. 신규 Cell 생성 확정

19. Colony 상태 갱신
20. Global Statistics 계산

21. Protocol 성공/실패 판정
22. Observation 판정
23. Narrative 조건 판정
24. Event Flush
```

처리 원칙(정본 10.2):

- 치명 피해를 받은 세포는 같은 tick에 분열하지 못한다.
- 신규 세포는 다음 tick부터 행동한다.
- Protocol 성공/실패 판정은 사망 결과가 확정된 뒤(17단계 이후) 처리한다.
- 여러 세포가 같은 영양분을 두고 경쟁하면 배열 순서로 선착순 독점하지 않는다. 요청량을 모아 동시에, 공급을 넘으면 비례로 나눈다.
- 위협 개체도 같은 순서를 따른다. 감지·이동은 5~8단계, 공격·섭취는 10~12단계, 증식과 사망은 16~18단계에서 세포와 함께 처리한다.

---

## 5. 시간과 Pause

- Briefing은 무제한·읽기 전용 pause 상태다.
- Preparing은 live-planning 상태다. simulation과 Preparation Timer가 진행되며 RESEARCH/MAIL/ANALYSIS/SYSTEM 화면 전환으로 멈추지 않는다.
- Preparing에서는 일반 manual pause를 받지 않는다.
- app background/OS interruption은 `systemSuspend`로 Preparing까지 포함해 pause하며 복귀 시 밀린 시간을 처리하지 않는다.
- Preparing 이외의 MAIL/RESEARCH/ANALYSIS/SYSTEM/GAME OVER는 기존 PauseSource를 사용한다.
- SimulationCommand는 pause 중 거부하고 Transaction은 상태별 허용 규칙을 따른다.
- 개발 빌드는 헤드리스 실행과 배속을 지원한다.

## 6. 이벤트 목록

`GameEvents`(명세서 #29)의 이름은 임의로 바꾸지 않는다. 새 이벤트만 추가한다.

| 이벤트 | 발행 주체 |
|---|---|
| `cell:born` | culture 계열(신규 Cell 생성 확정, 18단계) |
| `cell:divided` | DivisionSystem(분열 완료 시, #14.3) |
| `cell:died` | DeathSystem(사망 확정 시, #15) |
| `protocol:preparing` | ProtocolService/Runner(Preparing 진입 시) |
| `protocol:started` | ProtocolService/Runner(`transitionToRunning` 시) |
| `protocol:completed` | ProtocolService/Runner(Completed 전환 시) |
| `protocol:failed` | ProtocolService/Runner(Failed 전환 시) |
| `research:unlocked` | ResearchService(해금 확정 시) |
| `mail:received` | MailService |
| `narrative:flagChanged` | NarrativeService |
| `checkpoint:created` | CheckpointService(Preparing 진입 시 생성 직후) |
| `culture:reseeded` | culture 계열(Running이 아닐 때 전멸 후 재시딩, #22.3) |
| `observation:recorded` | ObservationService |
| `discovery:recorded` | DiscoveryService |

사용 기준(#30): 같은 feature 내부, 즉시 결과가 필요한 계산, 명확한 단일 책임 관계는 직접 함수 호출을 쓴다. 다른 feature가 선택적으로 반응하거나, 발신 시스템이 수신자를 몰라도 되거나, 통계/메일/스토리/오디오처럼 부가 반응일 때는 이벤트를 쓴다.

---

## 7. 저장

저장은 네 계층으로 나눈다.

| 계층 | 역할 |
|---|---|
| Campaign Save | 활성 캠페인 1개, CONTINUE 대상 |
| Rolling Autosave | 최근 정상본 3세대, 손상 시 fallback |
| Protocol Checkpoint | Preparing 진입 시 생성하는 별도 회귀 지점 |
| ProfileData | 엔딩 기록과 모드 해금 |

- saveNow는 슬롯 번호를 받지 않는다.
- autosave는 일관된 state boundary에서 확정하고 연속 요청은 합친다.
- Failed 상태를 정상 autosave로 덮어쓰지 않는다.
- 이전 protocol checkpoint는 SYSTEM → RECOVERY에서 선택할 수 있다.
- 과거 checkpoint 복원 시 그 지점 뒤의 campaign/autosave/checkpoint를 폐기한다.
- 저장 실패 시 직전 정상본을 보존한다.
- SaveData 변경은 version + migration + test를 동반한다.

## 8. 데이터 디렉터리 규칙

- `src/data/` 구성: `research/`, `protocols/`, `threats/`, `mail/`, `narrative/`, `balance/`(#3).
- 숫자 하드코딩 금지: 성장 시간, 연구 비용, 적 증식률, 산소 소비량, 신호 반경, 프로토콜 시간, 메일 발생 조건 등은 모두 데이터 파일에서 관리한다(#2.4).
- 시작 시 데이터 검증: 존재하지 않는 prerequisite, 순환 prerequisite, 존재하지 않는 Trait ID, 음수 비용, 중복 ID, 존재하지 않는 Mail trigger, Protocol phase 시간 역전 등을 게임 시작 시 검사한다. 가능하면 Zod 같은 런타임 스키마 도구를 쓴다(#41).
- 현지화 키: 화면에 나오는 글은 코드와 콘텐츠 데이터에 직접 적지 않고 키만 두며, 글은 `Game/Locale/<언어 코드>.json`에서 가져온다. 기준 파일은 `en.json`이고 새 언어는 JSON 파일 하나 추가로 끝나야 한다. 자리표시자는 `{count}` 형식이며 이름은 바꾸지 않는다. 없는 키는 조용히 빈 문자열로 넘기지 않고 개발 중에는 키 이름을 노출하며 로그를 남긴다(#19.2). 콘텐츠가 가리키는 키가 `en.json`에 있는지는 시작 시 데이터 검증(#41)에서, 언어 파일 사이의 키·자리표시자 일치는 `node Locale/check.mjs`로 검사한다(`Game/Locale/README.md`). Domain 계층은 `Localizer`를 쓰지 않는다.

---

## 9. 새 기능 추가 절차

**새 연구**(#46): 1) `data/research/...`에 정의 추가 2) 필요하면 Trait 데이터 추가 3) 완전히 새 행동이면 관련 system만 확장 4) 테스트 추가. ResearchService 자체는 보통 수정하지 않는다.

**새 적**(#47): 1) `data/threats/`에 파라미터 추가 2) 기존 시스템(BacteriaSystem 등)이 지원하는 파라미터를 우선 사용 3) Protocol data 반영 4) 테스트 추가. 새로운 행동 메커니즘이 있을 때만 System을 확장한다.

**새 메일**(#48): 1) mail data 작성 2) trigger condition 지정 3) 필요하면 narrative flag data 추가. NarrativeService 내부에 `if (mailId === ...)` 조건문을 계속 늘리지 않는다.

**새 플레이어 액션**(#49): 1) `PlayerAction` 타입 추가 2) 전용 Service 또는 CultureService 확장 3) Input/UI 연결 4) 밸런스 데이터 추가 5) 테스트 추가. UI가 직접 필드를 수정하지 않는다.

콘텐츠 하나(연구/메일/기존 행동을 쓰는 적 변형/프로토콜 데이터) 추가 시 기존 시스템 **0~2개 수정**은 강제 규칙이 아니라 구조 건강도를 보는 지표다. 새로운 근본 메커니즘일 때는 Domain/Save/UI/Test/System이 함께 바뀌는 것도 정상이며 실패로 취급하지 않는다(#63, 정본 31).

---

## 10. 금지 패턴 (#50)

1. God Manager — 세포·연구·메일·세이브·프로토콜·UI·사운드를 한 클래스에 몰아넣지 않는다.
2. Renderer에서 로직 처리 — 사망·감염 등 판정을 Renderer에 넣지 않는다.
3. UI가 상태를 직접 변경 — Command/Transaction을 거친다.
4. Boolean 상태 난립 — 한 축의 상태는 enum/state machine으로 묶는다.
5. Magic Number — 밸런스 값을 코드에 직접 비교하지 않는다.
6. 타입 오류를 `any`로 덮기.
7. 기능 하나 추가하며 대규모 리팩터링 — 리팩터링은 별도 작업으로 분리한다.

---

## 11. 테스트

Vitest를 쓴다(정본 30). 게임 규칙을 테스트하고 렌더링은 테스트하지 않는다(#39). 에너지, Movement, Research, Protocol, 발견/Data, Pause/UI, Division, Trait, Narrative, Save, Performance 영역의 구현 전 필수 시나리오는 명세서 #39와 정본 35에 있다. 통합 테스트로 최소 두 흐름을 자동화한다: 새 게임→세포 생성→먹이 공급→연구 해금→특성 활성→Preparing(Checkpoint 생성)→시작→완료→보상→메일→저장→로드, 그리고 Preparing(Checkpoint 생성)→시작→실패→GAME OVER→Checkpoint 전체 로드→Preparing(#40).

단계별 테스트는 `Game/Docs/current/DEVELOPMENT_PIPELINE.html`의 단계마다 다음 3차 절차를 순서대로 거치고 사람이 승인해야 다음 단계로 간다(정본 30.2). 1) 에이전트 배속 테스트: 자동 테스트 전체와, 화면 없이 빠르게 돌려 규칙을 확인하는 헤드리스 시나리오. 2) 브라우저 배속 테스트: 메인 에이전트가 Chrome에 실제 화면을 띄워 배속으로 돌리며 화면·콘솔 오류·조작을 확인한다. 3) 사람 테스트: `Game/Docs/tests/`의 체크리스트로 사람이 직접 판정한다. 재미와 느낌에 대한 관문은 사람만 판정한다. 이를 위해 시뮬레이션은 화면 없이 돌 수 있어야 하고 개발 빌드에는 배속 설정이 있어야 하며, 배속은 tick 하나의 계산을 바꾸지 않는다.


## 12. 모바일 Viewport 경계

- 기준 디자인: 1080×1920 세로.
- 논리 화면: 216×384.
- PixelCanvas는 고정 논리 화면만 책임진다.
- ResponsiveShell은 safe area, 화면비, 중앙 배치, 외부 CRT 케이스 여백을 책임진다.
- CrtDisplay는 shell 안의 게임 화면에 후처리를 적용한다.
- 화면비는 simulation chamber 좌표나 게임 규칙을 바꾸지 않는다.
- InputRouter는 physical → shell local → inverse CRT → logical 순으로 좌표를 변환한다.
