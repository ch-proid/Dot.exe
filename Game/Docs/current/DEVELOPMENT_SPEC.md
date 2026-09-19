# Dot.exe 개발 명세서

> 버전: 0.3 (정본 `CORE_GAME_RULES.md` v1.4 반영)  
> 대상: TypeScript 기반 구현 (Vite, Vitest)  
> 출시 대상: Google Play · App Store. 개발 중에는 데스크톱 브라우저에서 실행하고 출시할 때 Capacitor로 감싼다. 세로 고정, 터치 입력(마우스도 같은 경로). (정본 30절)  
> 렌더링: 기준 디자인 1080×1920, 논리 화면 216×384. PixelCanvas → ResponsiveShell(safe area/화면비) → CrtDisplay(WebGL) 순서로 표시하며 터치는 역변환해 논리 좌표로 보낸다.
> 목적: 바이브코딩 환경에서도 수정·기능 추가 시 기존 시스템이 연쇄적으로 깨지지 않도록 구조를 먼저 고정한다.  
> 문서 간 규칙이 어긋날 경우의 우선순위는 `CORE_GAME_RULES.md` 1절을 따른다. 본문의 “정본”은 `CORE_GAME_RULES.md`다.  
> 시뮬레이션은 화면 없이 돌릴 수 있어야 하고 개발 빌드에는 배속 설정을 둔다. 단계별 테스트 절차는 정본 30.2절을 따른다.

---

# 1. 문서 목적

작품명과 극중 플레이어가 실행하는 배양 시스템 소프트웨어명은 모두 `Dot.exe`다.

이 문서는 Dot.exe의 구현 경계를 정의한다.

목표는 다음과 같다.

1. 기능 추가 시 기존 시스템 수정 범위를 작게 유지한다.
2. AI가 관련 없는 파일까지 대규모로 고치지 않도록 책임 경계를 명확히 한다.
3. 게임 로직과 UI, 렌더링, 저장을 분리한다.
4. 밸런스와 콘텐츠를 코드 밖 데이터로 분리한다.
5. 핵심 규칙을 테스트 가능하게 만든다.
6. 세포 수가 증가해도 성능을 유지할 수 있는 구조를 사용한다.
7. 저장 데이터 변경 시 마이그레이션이 가능하게 한다.

---

# 2. 구현 원칙

## 2.1 Feature 단위 모듈화

게임 기능을 기술 계층보다 도메인 단위로 나눈다.

예:

- culture
- research
- protocol
- mail
- narrative
- resources
- save

한 기능 수정 시 가능한 한 해당 feature 내부에서 끝나야 한다.

## 2.2 UI는 게임 상태를 직접 수정하지 않는다

금지:

```ts
button.onclick = () => {
  game.energy -= 100;
  cell.speed += 0.2;
}
```

허용:

```ts
button.onclick = () => {
  researchService.purchase(researchId);
}
```

실제 검증과 상태 변경은 도메인 서비스가 담당한다.

## 2.3 렌더러는 규칙을 가지지 않는다

렌더링 레이어는 상태를 읽고 보여주기만 한다.

세포가 사망하는 조건, 감염되는 조건, 에너지가 생성되는 조건은 Renderer에 작성하지 않는다.

## 2.4 숫자 하드코딩 금지

다음은 모두 데이터 파일에서 관리한다.

- 성장 시간
- 연구 비용
- 적 증식률
- 산소 소비량
- 신호 반경
- 프로토콜 시간
- 메일 발생 조건

## 2.5 시스템 간 직접 의존 최소화

가능하면 Domain Event를 사용한다.

예:

```ts
eventBus.emit("cell:divided", payload);
eventBus.emit("protocol:completed", payload);
eventBus.emit("mail:received", payload);
```

## 2.6 기존 public contract를 함부로 바꾸지 않는다

공개 인터페이스 변경은 기능 추가보다 더 큰 작업으로 취급한다.

변경 시 호출 지점을 모두 점검하고 테스트를 갱신한다.

---

# 3. 권장 프로젝트 구조

```text
src/
├─ app/
│  ├─ GameApp.ts
│  ├─ bootstrap.ts
│  └─ AppContext.ts
│
├─ core/
│  ├─ events/
│  │  ├─ EventBus.ts
│  │  ├─ events.ts
│  │  └─ eventTypes.ts
│  ├─ time/
│  │  ├─ GameClock.ts
│  │  ├─ FixedStepLoop.ts
│  │  └─ PauseController.ts
│  ├─ random/
│  │  ├─ RandomSource.ts
│  │  └─ SeededRandom.ts
│  │     (Gameplay/Presentation 두 인스턴스로 생성, #36 참고)
│  └─ math/
│     ├─ Vec2.ts
│     └─ SpatialHash.ts
│
├─ features/
│  ├─ culture/
│  │  ├─ domain/
│  │  │  ├─ Cell.ts
│  │  │  ├─ LifeMode.ts
│  │  │  ├─ CellTrait.ts
│  │  │  ├─ Colony.ts
│  │  │  └─ CultureState.ts
│  │  ├─ systems/
│  │  │  ├─ CellMovementSystem.ts
│  │  │  ├─ MetabolismSystem.ts
│  │  │  ├─ DivisionSystem.ts
│  │  │  ├─ StressSystem.ts
│  │  │  ├─ DeathSystem.ts
│  │  │  ├─ InfectionSystem.ts
│  │  │  ├─ ColonySystem.ts
│  │  │  └─ SignalSystem.ts
│  │  ├─ fields/
│  │  │  ├─ NutrientField.ts
│  │  │  ├─ ToxinField.ts
│  │  │  └─ SignalField.ts
│  │  ├─ CultureService.ts
│  │  ├─ culture.events.ts
│  │  └─ culture.types.ts
│  │
│  ├─ research/
│  │  ├─ ResearchService.ts
│  │  ├─ TraitLoadoutService.ts
│  │  ├─ research.types.ts
│  │  └─ research.events.ts
│  │
│  ├─ protocols/
│  │  ├─ ProtocolService.ts
│  │  ├─ ProtocolRunner.ts
│  │  ├─ protocol.types.ts
│  │  └─ protocol.events.ts
│  │
│  ├─ threats/
│  │  ├─ ThreatFactory.ts
│  │  ├─ bacteria/
│  │  ├─ virus/
│  │  ├─ parasite/
│  │  ├─ fungus/
│  │  └─ environmental/
│  │
│  ├─ resources/
│  │  ├─ ResourceService.ts
│  │  └─ resource.types.ts
│  │
│  ├─ mail/
│  │  ├─ MailService.ts
│  │  ├─ MailInbox.ts
│  │  ├─ mail.types.ts
│  │  └─ mail.events.ts
│  │
│  ├─ narrative/
│  │  ├─ NarrativeService.ts
│  │  ├─ NarrativeCondition.ts
│  │  ├─ EmergenceService.ts
│  │  ├─ ObservationService.ts
│  │  ├─ DiscoveryService.ts
│  │  └─ narrative.types.ts
│  │
│  └─ save/
│     ├─ SaveService.ts
│     ├─ SaveData.ts
│     ├─ CheckpointService.ts
│     ├─ ProfileData.ts
│     ├─ migrations/
│     └─ serializers/
│
├─ data/
│  ├─ research/
│  │  ├─ metabolism.ts
│  │  ├─ reproduction.ts
│  │  ├─ motility.ts
│  │  ├─ defense.ts
│  │  ├─ colony.ts
│  │  └─ signaling.ts
│  ├─ protocols/
│  ├─ threats/
│  ├─ mail/
│  ├─ narrative/
│  └─ balance/
│
├─ presentation/
│  ├─ render/
│  │  ├─ CultureRenderer.ts
│  │  ├─ FieldRenderer.ts
│  │  └─ EffectsRenderer.ts
│  ├─ ui/
│  │  ├─ screens/
│  │  ├─ panels/
│  │  └─ components/
│  └─ audio/
│     └─ AudioService.ts
│
├─ input/
│  ├─ InputRouter.ts
│  └─ actions.ts
│
└─ tests/
   ├─ unit/
   ├─ integration/
   └─ fixtures/
```

---

# 4. AppContext

전역 싱글턴을 남발하지 않는다.

의존성은 AppContext를 통해 한 번 조립한다.

```ts
export interface AppContext {
  events: EventBus;
  clock: GameClock;
  pause: PauseController;

  gameplayRandom: RandomSource;
  presentationRandom: RandomSource;

  culture: CultureService;
  research: ResearchService;
  protocols: ProtocolService;
  resources: ResourceService;
  mail: MailService;
  narrative: NarrativeService;
  observations: ObservationService;
  save: SaveService;
  checkpoints: CheckpointService;
}
```

`gameplayRandom`과 `presentationRandom`은 서로 다른 시드를 가진 별도 인스턴스다. 연출용 난수가 게임 결과를 바꾸지 않도록 분리한다(#36 참고).

각 feature 내부에서는 필요한 인터페이스만 주입받는다.

---

# 5. 시뮬레이션 업데이트 구조

세포 수가 많아질 가능성이 있으므로 Variable Delta만 사용하는 구조를 피한다.

권장:

- 화면 렌더링: 가변 프레임
- 게임 로직: 고정 스텝

예:

```text
Fixed simulation step: 1/20 sec (초기값)
Render: requestAnimationFrame
```

20Hz는 초기값이며 성능 검증 후 조정할 수 있다. 실제 값은 BalanceData에서 관리한다.

한 프레임에서 여러 tick이 밀려 있을 때 따라잡는 최대 step 수도 설정값으로 둔다. 무한정 따라잡지 않는다.

창이 비활성 상태가 되면 시뮬레이션은 Pause한다. 기본 캠페인에서는 오프라인 진행(창을 닫은 동안 시간이 흐르는 처리)을 사용하지 않는다.

## 5.1 Simulation Time과 UI Transaction 처리 분리

- Briefing: pause, 조회 전용.
- Preparing: live planning. simulation과 Preparation Timer가 계속 돌고 modal 화면 전환은 pause source를 추가하지 않는다. manual pause도 비활성화한다.
- Running: 연구·Trait 변경 거부.
- Preparing 이외의 MAIL/RESEARCH/ANALYSIS/SYSTEM/GAME OVER는 기존 pause source를 쓴다.
- app background/OS interruption은 `systemSuspend`로 모든 시간을 멈추며 복귀 시 밀린 시간을 처리하지 않는다.

PauseController는 source 집합을 유지한다.

# 6. 시뮬레이션 시스템 실행 순서

한 tick에서 순서를 고정한다.

```text
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

순서는 임의로 바꾸지 않는다.

시스템 간 결과 재현성을 위해 문서화한다.

## 6.1 중요한 처리 원칙

- 치명 피해를 받은 세포는 같은 tick에 분열하지 못한다.
- 신규 세포는 다음 tick부터 행동한다.
- Protocol 성공/실패 판정은 해당 tick의 피해·사망 결과가 확정된 뒤(17단계 이후) 처리한다.
- 여러 세포가 같은 영양분을 먹으려 할 때 배열 순서로 선착순 독점하지 않는다.
- 경쟁 자원은 요청량을 모아 동시에 분배한다. 요청이 공급을 넘으면 요청량에 비례해 나눈다.
- 위협 개체도 같은 순서를 따른다. 감지·이동은 5~8단계, 공격·섭취는 10~12단계, 증식과 사망은 16~18단계에서 세포와 함께 처리한다.

---

# 7. Cell 데이터 모델

권장 예시:

```ts
export interface Cell {
  id: CellId;

  position: Vec2;
  velocity: Vec2;
  wander: WanderState; // 부드러운 방향 변화를 위한 내부 상태

  energy: number;
  age: number;
  stress: number;

  health: number; // 0~1, 손상 정도
  pendingDeath: boolean;

  lifeMode: LifeMode;

  lastDivisionTick: number;
  divisionProgress: number; // Dividing 상태일 때 진행도

  colonyId?: ColonyId; // MVP 이후. MVP에서는 사용하지 않는다.
  infection?: InfectionState;
}
```

`activeTraits`는 Cell 필드로 두지 않는다. MVP에서 Trait은 배양체 전체가 공유하는 전역 Loadout이다(#20 참고).

초기에는 class보다 단순 데이터 객체를 우선해도 된다.

단, 상태 변경은 System 또는 Service를 통해 수행한다.

---

# 8. LifeMode

Boolean 다중 조합을 피한다.

금지:

```ts
isDead
isDividing
isDormant
isInfected
isStressed
```

권장:

```ts
export enum LifeMode {
  Active = "active",
  Dividing = "dividing",
  Dormant = "dormant",
  Dead = "dead",
}
```

`Damaged`는 별도 LifeMode로 두지 않는다. 손상 정도는 `Cell.health`(0~1)로 표현한다.

감염은 `LifeMode`와 독립된 축이다. `Cell.infection`으로 따로 관리하며, 다음과 같은 조합이 모두 가능하다.

```text
Dormant + Low Health + Infected
Dividing + Damaged(낮은 Health)
Active + Infected
```

LifeMode와 손상 상태, 감염 상태가 동시에 존재할 수 있기 때문에 세 축을 분리한다.

---

# 9. 이동 모델

## 9.1 타일 없음

Cell 위치는 연속 좌표다.

```ts
type Vec2 = {
  x: number;
  y: number;
}
```

## 9.2 Steering Force

최종 이동력:

```text
F =
  wander
+ nutrientAttraction
+ hazardAvoidance
+ separation
+ adhesion
+ threatPursuit
+ signalInfluence
+ boundaryRepulsion
```

각 요소는 독립 함수로 구현한다.

adhesion은 아직 같은 군집(Colony)에 속하지 않은 세포끼리도 작용한다. 같은 군집 세포만 끌어당기면 첫 군집이 만들어지지 않는다(#18 참고).

```ts
interface SteeringContext {
  cell: Cell;
  nearbyCells: readonly Cell[];
  nearbyThreats: readonly Threat[];
  fields: CultureFields;
}
```

## 9.3 힘 가중치

가중치는 Trait과 BalanceData에서 계산한다.

코드에 직접 숫자를 박지 않는다.

## 9.4 속도 제한

항상 다음을 clamp한다.

```ts
speed <= maxSpeed
acceleration <= maxAcceleration
```

## 9.5 배양 공간 경계

배양 공간은 닫힌 사각형 Chamber다. 반대편으로 Wrap하지 않는다.

경계 처리는 세 가지를 함께 사용한다.

- Soft Repulsion: 경계에 가까워질수록 `boundaryRepulsion` 힘이 커진다.
- Hard Clamp: 그래도 경계를 넘으면 위치를 강제로 경계 안쪽으로 고정한다.
- Boundary Normal 방향 속도 감소: 경계에 부딪힌 세포의 법선 방향 속도 성분을 줄인다.

경계 계수는 BalanceData에서 관리한다. 이 규칙은 후반 "경계 발견" 서사와 양립해야 하므로 임의로 바꾸지 않는다.

---

# 10. Spatial Hash

모든 세포끼리 거리 계산하면 O(n²)가 된다.

따라서 근접 검색은 Spatial Hash 또는 Uniform Grid를 사용한다.

이 Grid는 **이동 타일이 아니라 계산 최적화용**이다.

예:

```ts
interface SpatialQuery {
  findCellsInRadius(position: Vec2, radius: number): readonly CellId[];
  findThreatsInRadius(position: Vec2, radius: number): readonly ThreatId[];
}
```

---

# 11. 환경 필드

영양, 독성, 신호는 개체 객체 수천 개로 구현하지 않는다.

저해상도 스칼라 필드를 사용한다.

예:

```ts
interface ScalarField {
  sample(position: Vec2): number;
  gradient(position: Vec2): Vec2;
  add(position: Vec2, amount: number, radius: number): void;
  update(dt: number): void;
}
```

MVP Field:

- nutrient
- toxin
- signal (Alarm Signal 전용)

oxygen, defenseSignal, attractSignal, retreatSignal 등은 MVP 이후다.

벽(경계)은 no-flux로 처리한다. 필드가 경계를 넘어 확산하지 않는다.

확산 계수와 감쇠 계수는 BalanceData에서 관리한다.

세포 이동은 `gradient()`를 사용한다.

---

# 12. Movement System 책임

`CellMovementSystem`은 이동만 담당한다.

하지 않는 일:

- 에너지 소비 판정
- 감염 판정
- 사망 처리
- 연구 해금
- 렌더링

에너지 소모는 Movement 결과를 MetabolismSystem이 참고한다.

---

# 13. Metabolism System

입력:

- 세포
- 지역 영양 농도
- 활성 특성(현재 전역 Trait Loadout)

출력:

- 에너지 변화
- 영양 소비

공식은 별도 BalanceData에서 관리한다.

## 13.1 에너지 흐름

"내부 저장 한도를 넘은 에너지만 연구 자원으로 회수"하는 방식은 사용하지 않는다. 분열과 연구 자원이 서로 막히는 문제를 피하기 위해 **대사 순생산을 분배하는 방식**을 사용한다.

```text
NUTRIENT
   ↓
Gross Metabolic Output
   ↓
기초 생존 비용
- 유지
- 이동
- 활성 행동
   ↓
Net Metabolic Output
   ↓
 ┌────────────────────┐
 │                    │
 ▼                    ▼
Cell Internal Energy  ENERGY
```

## 13.2 별도 관리 값

다음 값은 서로 다른 개념이며 서로 같은 값으로 쓰지 않는다. 초기 밸런스 값은 BalanceData에서 관리한다.

```text
survivalReserve
divisionRequirement
maximumStorage
systemHarvestRatio
```

## 13.3 생존 우선 규칙

세포 내부 에너지가 `survivalReserve`보다 낮으면 ENERGY 회수를 중단한다.

```text
Cell Energy < survivalReserve
→ 순생산 100%를 세포 내부에 배정

Cell Energy >= survivalReserve
→ 순생산 일부를 내부 저장, 일부를 ENERGY로 배정
```

`Energy Storage` 연구는 `maximumStorage`만 증가시킨다. ENERGY 회수 기준이나 회수 비율을 직접 올리지 않는다.

## 13.4 경계 경우

- 내부 에너지가 `maximumStorage`에 찬 세포도 섭취와 대사를 계속한다.
- 저장하지 못한 내부 몫은 버린다. ENERGY로 돌리지 않는다. 돌리면 "넘친 만큼 회수" 방식이 되살아나 Energy Storage 구매가 ENERGY 생산을 깎는 문제가 생긴다.
- 순생산이 음수면 내부 에너지에서 뺀다.
- Trait 변경으로 `maximumStorage`가 줄면 넘는 몫은 바로 사라진다.

## 13.5 굶주림

내부 에너지가 0이면 Health가 BalanceData가 정한 속도로 줄어든다. Health가 0이 되면 `Cell.pendingDeath`가 `true`가 된다.

에너지를 되찾으면 Health는 천천히 회복한다.

---

# 14. Division System

분열 조건:

```ts
interface DivisionRule {
  divisionRequirement: number; // 분열 시작에 필요한 내부 에너지
  divisionCost: number;
  maturityAge: number;
  divisionCooldown: number;
  divisionDuration: number;
  stressLimit: number;
}
```

`maxLocalDensity`는 기본 규칙에 넣지 않는다. 고밀도에서 분열을 억제하는 것은 Density Sensing 연구가 제공하는 behavior의 파라미터다(#16 참고). 연구 없이 과밀해진 군집은 영양 경쟁으로 대가를 치른다.

## 14.1 분열 시간 조건

첫 분열과 그 뒤의 분열을 가르지 않고 항상 두 조건을 함께 본다.

```text
age >= maturityAge
AND
currentTick - lastDivisionTick >= divisionCooldown
```

최초 세포의 `lastDivisionTick`은 생성된 tick이다. `lastDivisionTick`은 분열이 완료된 시점에 기록한다.

## 14.2 개체 수 상한

개체 수 상한은 분열을 시작할 때 `Viable Cell 수 + Dividing 상태 세포 수`로 검사한다. 비용만 내고 자식이 생기지 않는 일이 없게 한다.

## 14.3 분열 절차

1. 조건(에너지, 시간, 스트레스, 개체 수 상한)을 모두 만족하면 분열을 시작하고 `lifeMode`를 Dividing으로 바꾼다.
2. 시작 시점에 `divisionCost`를 예약·차감한다. 도중 사망하면 반환하지 않는다.
3. `divisionDuration` 동안 `divisionProgress`가 진행된다.
4. 완료 시 남은 내부 에너지를 부모와 자식이 반씩 나눈다. 분열 시 에너지가 새로 생성되지 않는다.
5. `parent.age`는 유지하고 `child.age = 0`으로 설정한다.
6. `parent.lastDivisionTick`과 `child.lastDivisionTick`을 모두 `now`로 기록한다.
7. 부모와 자식 위치를 분리한다.
8. `cell:divided` 이벤트를 발행한다.

분열 도중 사망하면 비용을 반환하지 않고 자식도 생성되지 않는다.

감염 상속 규칙은 감염 종류별 데이터로 따로 정의한다.

---

# 15. Death System

사망은 한 곳에서만 처리한다.

PendingDeath 표시와 사망 확정을 분리한다(tick 순서 #6의 15단계와 17단계). Health가 0이 되는 등 사망 조건이 발생하면 먼저 `pendingDeath`를 `true`로 표시하고, 분열 판정(16단계)에서 이 개체를 제외한 뒤, 17단계에서 실제 사망을 확정한다.

사망 원인은 enum으로 남긴다.

```ts
export enum CellDeathCause {
  Starvation,
  Toxin,
  Infection,
  ThermalShock,
  Purge,
  Age,
  ProgrammedDeath,
}
```

이벤트:

```ts
type CellDiedEvent = {
  cellId: CellId;
  cause: CellDeathCause;
  position: Vec2;
}
```

Narrative나 통계가 직접 Cell을 감시하지 않고 이벤트를 구독한다.

---

# 16. Trait 시스템

MVP에서는 **배양체 전체가 하나의 전역 Trait Loadout을 공유**한다. 세포별로 다른 Trait을 갖지 않는다.

```text
CULTURE GENOME
      ↓
Trait Loadout
      ↓
모든 일반 세포
```

개별 세포 차이(mutation, role, phenotypeModifier)는 MVP 이후 구조로 추가한다.

Trait은 코드 분기문 남발이 아니라 데이터 + 태그 기반으로 만든다.

예:

```ts
export interface TraitDefinition {
  id: CellTraitId;
  category: TraitCategory;

  expressionLoad: number;

  modifiers?: TraitModifier[];
  behaviors?: TraitBehaviorId[];

  prerequisites: ResearchId[];
}
```

`behaviors`는 시스템 기능을 켜는 용도다.

예:

```text
chemotaxis
hazard_avoidance
phagocytosis
collective_defense
signal_relay
```

---

# 17. Modifier 모델

수치 보정은 표준화한다.

```ts
export interface TraitModifier {
  stat: CellStat;
  operation: "add" | "multiply";
  value: number;
}
```

예:

```ts
{
  stat: "movement.maxSpeed",
  operation: "multiply",
  value: 1.15
}
```

세포의 유효 능력치는 매 계산마다 `기본값 + 현재 Trait Loadout`으로 다시 구한다. Modifier를 세포에 누적 적용하지 않는다. 계산 순서는 `add` 연산을 먼저 모두 적용한 뒤 `multiply` 연산을 적용한다.

각 시스템이 임의로 Trait 이름을 검사하는 코드를 최소화한다.

---

# 18. 행동형 Trait

새 행동은 `behavior id`를 통해 각 시스템에서 활성화한다.

예:

`Phagocytosis`는 DefenseSystem이 인식한다.

`Chemotaxis`는 MovementSystem이 인식한다.

`CollectiveMemory`는 Narrative/Colony 관련 시스템이 인식한다.

한 Trait 때문에 여러 feature를 동시에 직접 수정해야 한다면 구조를 다시 검토한다.

---

# 19. Research 데이터 구조

```ts
export interface ResearchDefinition {
  id: ResearchId;
  category: ResearchCategory;

  titleKey: LocaleKey;        // 예: "trait.chemotaxis.name"
  descriptionKey: LocaleKey;  // 예: "trait.chemotaxis.effect"

  energyCost: number;
  dataCost: number;

  prerequisites: ResearchId[];

  unlockTraits?: CellTraitId[];
  unlockActions?: PlayerActionId[];
  unlockProtocols?: ProtocolId[];
}
```

ResearchService는 해금과 비용 검증만 담당한다.

세포 행동은 ResearchService 안에 작성하지 않는다.

## 19.1 구매 제한과 보상 지급

연구 구매는 `Running` 상태에서만 거부한다. Idle, Preparing, Completed에서는 허용한다. 거부 사유는 `PROTOCOL_RUNNING`이다(#43 참고).

Chemotaxis처럼 구매가 아니라 프로토콜 완료 보상으로 연구 완료 상태를 바로 지급하는 경우가 있다. 이를 위해 `ProtocolReward`에 연구 지급 종류를 둔다.

```ts
type ProtocolReward =
  | { type: "resource"; resource: keyof ResourceState; amount: number }
  | { type: "grantResearch"; researchId: ResearchId; activateTraits: boolean };
```

## 19.2 현지화

화면에 나오는 글은 코드와 콘텐츠 데이터에 직접 적지 않는다. 데이터는 키만 갖고, 글은 `Game/Locale/<언어 코드>.json`에서 가져온다(정본 30.1절).

```ts
type LocaleKey = Brand<string, "LocaleKey">;

interface Localizer {
  t(key: LocaleKey, params?: Readonly<Record<string, string | number>>): string;
}
```

- 기준 파일은 `en.json`이다. 새 언어는 JSON 파일 하나를 더하는 것으로 끝나야 한다.
- 자리표시자는 `{count}` 형식이다.
- 없는 키는 조용히 빈 문자열로 넘기지 않는다. 개발 중에는 키 이름을 그대로 보여주고 로그를 남긴다.
- 데이터 검증(#41)에서 콘텐츠 데이터가 가리키는 키가 `en.json`에 있는지 검사한다. 언어 파일끼리의 키·자리표시자 일치는 `node Locale/check.mjs`로 검사한다.
- 게임 규칙과 서사 조건은 언어와 무관하다. Domain 계층은 `Localizer`를 쓰지 않으며 `presentation/`만 쓴다.
- 폰트는 언어별로 지정할 수 있게 한다.

---

# 20. Trait Loadout

해금과 발현을 분리한다.

```ts
interface TraitLoadout {
  activeTraitIds: CellTraitId[];
  expressionLoad: number;
  safeExpressionLimit: number;
  hardExpressionLimit: number;
}
```

발현 부하는 두 단계 한도로 관리한다.

- `expressionLoad`가 `safeExpressionLimit` 이하면 정상 작동한다.
- `safeExpressionLimit`을 초과하면 활성은 허용하되 과발현 패널티가 발생한다. 패널티는 기초 대사 부담, 스트레스 발생, 분열 비용, 유지 효율 계열에 적용하며 정확한 계수는 BalanceData에서 관리한다.
- `hardExpressionLimit`을 초과하는 조합은 활성화할 수 없다.

`TraitLoadoutService`는 Safe Limit 초과를 거부하지 않는다. Hard Limit 초과만 거부한다.

Trait Loadout 변경은 `Running` 상태에서만 거부한다. 그 밖의 상태(Idle, Preparing, Completed)에서는 허용한다. 실험 도중 상황을 보고 생산 특성을 방어 특성으로 즉시 갈아끼우는 플레이를 막기 위한 규칙이다.

활성 변경은 TraitLoadoutService가 검증한다.

---

# 21. Resources

```ts
export interface ResourceState {
  energy: number;
  data: number;
  nutrientReserve: number;
}
```

외부 시스템은 값을 직접 변경하지 않는다.

```ts
resourceService.trySpend(cost)
resourceService.add(type, amount, reason)
```

`reason`을 기록하면 디버깅이 쉬워진다.

## 21.1 발견 기록과 DATA 보상

발견은 종류별로 한 번만 기록하고, 기록하는 순간 DATA를 지급한다.

```ts
interface DiscoveryRecord {
  observed: boolean;
}
```

미확정(Pending) DATA는 두지 않는다.

획득 후보(MVP):

- 새 위협 최초 관찰
- 새 행동 최초 관찰
- Protocol 성공

관찰하거나 성공하는 순간 바로 지급한다. 실패해서 체크포인트를 불러오면 발견 기록과 DATA가 함께 되돌아가므로, 다시 하면서 다시 발견하고 다시 받는다(#33 참고). 능동적인 Analysis 미니 시스템은 MVP 이후로 미룬다.

`DiscoveryService`가 발견 기록을 관리한다.

---

# 22. Protocol 시스템

Protocol은 데이터 기반으로 정의한다.

```ts
export interface ProtocolDefinition {
  id: ProtocolId;

  preparationDuration: number;
  activeDuration: number;

  objectives: ProtocolObjective[];
  phases: ProtocolPhase[];

  rewards: ProtocolReward[];

  nextProtocolIds?: ProtocolId[];
}
```

## 22.1 Protocol 상태 흐름

```text
Idle → Briefing → Preparing → Running → Completed
                               ↘ Failed → GAME OVER → Checkpoint Load → Preparing
```

다음 프로토콜이 있으면 Completed 뒤 Briefing으로 간다. Briefing은 무제한·읽기 전용이고 `beginPreparation()`이 Preparing으로 전환한다. Preparing 진입 시 checkpoint 생성 후 timer를 시작한다. 수동/자동 시작은 모두 `transitionToRunning()`을 쓴다.

## 22.2 체크포인트

프로토콜이 `Preparing`에 들어가는 순간 정확히 한 번 체크포인트를 생성한다. 준비 타이머가 돌기 시작하는 시점이다.

체크포인트는 그 시점의 전체 저장 데이터이며 일반 저장과 같은 형식을 쓴다(#33 참고). 이전 프로토콜의 체크포인트도 지우지 않고 남겨 둔다.

`Running`에서 실패하면 GAME OVER 화면으로 전환하고(Pause), 체크포인트를 통째로 불러온 뒤 `Preparing`으로 돌아온다. 이때 준비 타이머는 처음부터 다시 돈다.

## 22.3 완료와 전멸 처리

`Completed`로 전환할 때 남아 있는 위협 개체를 제거한다. 이후 `nextProtocolIds`가 있으면 다음 프로토콜의 `Preparing`으로, 없으면 `Idle`로 전환한다.

`Running`이 아닐 때 Viable Cell이 0이 되면 게임 오버로 처리하지 않는다. `RESTORING LAST VIABLE SAMPLE` 연출과 함께 초기 세포 하나를 배양 공간 중앙에 다시 만든다. 자원과 연구는 그대로 둔다.

## 22.4 연구 구매·Trait 변경 제한

`Running` 상태에서만 금지한다. Idle, Preparing, Completed에서는 허용한다(#19.1, #20 참고).

---

# 23. Protocol Phase

복합 실험을 위해 단계 구조를 사용한다.

```ts
export interface ProtocolPhase {
  startAt: number;

  actions: ProtocolAction[];
}
```

예:

```ts
{
  startAt: 0,
  actions: [
    { type: "spawnThreat", threatId: "bacteria_rapid", count: 4 }
  ]
}
```

또는:

```ts
{
  startAt: 60,
  actions: [
    { type: "setNutrientReserveRefillRate", value: 0.45 }
  ]
}
```

MVP에 없는 값(Oxygen 등)을 예시로 쓰지 않는다. MVP에 있는 값(NUTRIENT RESERVE 보충 속도 등)만 사용한다.

## 23.1 ProtocolObjective

범용 `SURVIVAL RATE` 하나로 모든 실험을 판정하지 않는다. 목표를 명시적인 목적 단위로 나눈다.

```ts
type ProtocolObjective =
  | { type: "survivalDuration"; seconds: number }
  | {
      type: "populationRetention";
      referencePopulation: number;
      retentionRatio: number;
      absoluteMinimum: number;
      evaluationWindow: number;
    }
  | { type: "endPopulationAtLeast"; count: number }
  | { type: "avoidExtinction" }
  | { type: "maintainInfectionBelow"; ratio: number }
  | { type: "maintainEnvironment" } // MVP 이후. 환경값이 들어올 때 필드를 정한다.
  | { type: "observeAutonomousResponse" };
```

`populationRetention`의 필요 개체 수는 다음과 같이 계산한다.

```text
requiredPopulation =
  max(
    absoluteMinimum,
    ceil(referencePopulation * retentionRatio)
  )
```

기준은 프로토콜이 정한 `referencePopulation`만 쓴다. 시작 개체 수는 기준에 넣지 않는다. HUD에는 비율이 아니라 필요 개체 수를 보여준다(정본 5.2절, 27절).

"마지막 N초 유지" 같은 구간 조건은 평균이 아니라 해당 구간의 모든 판정 tick에서 조건을 충족해야 한다.

기본적으로 필수 목표는 `AND` 관계다. 보너스 목표만 선택 조건으로 둔다.

## 23.2 Viable Cell 판정

프로토콜 판정 전체에서 같은 판정 함수 하나를 사용한다.

포함: `LifeMode.Active`, `LifeMode.Dividing`, `LifeMode.Dormant`이면서 Health가 0보다 큰 개체.

제외: `pendingDeath`, `LifeMode.Dead`, 배양 공간에서 완전히 제거된 분석 샘플.

## 23.3 판정 순서

마지막 tick에 시간 종료와 전멸이 동시에 발생할 수 있다. 다음 순서로 처리한다.

```text
실패 조건 판정
→ 성공 조건 판정
→ 보상 지급
```

전멸 상태에서 시간만 끝났다고 성공하지 않는다.

---

# 24. Threat 인터페이스

각 위협은 공통 인터페이스를 가진다.

```ts
export interface Threat {
  id: ThreatInstanceId;
  typeId: ThreatTypeId;

  position?: Vec2;
  state: ThreatState;
}
```

위협 종류별 시스템을 분리할 수 있다.

예:

- BacteriaSystem
- VirusSystem
- ParasiteSystem
- FungusSystem
- EnvironmentThreatSystem

위협 개체도 세포와 같은 tick 순서를 따른다(#6 참고). 감지·이동은 5~8단계, 공격·섭취는 10~12단계, 증식과 사망은 16~18단계에서 세포와 함께 처리한다.

## 24.1 Rapid Bacteria (MVP)

Rapid Bacteria는 Nutrient를 먹어 증식하고 약한 Toxin을 낸다. 증식량과 Toxin 배출량 등 구체 파라미터는 `data/threats/`에서 관리한다.

---

# 25. Mail 시스템

메일 콘텐츠는 코드에 하드코딩하지 않는다.

```ts
export interface MailDefinition {
  id: MailId;
  senderKey: LocaleKey;   // 예: "sender.miller"
  subjectKey: LocaleKey;  // 예: "mail.culture01.subject"
  bodyKey: LocaleKey;     // 예: "mail.culture01.body"

  trigger: NarrativeConditionDefinition;
  priority: number;

  oneShot: boolean;
}
```

MailService 역할:

- 받은 메일 관리
- 읽음 상태 관리
- 신규 메일 이벤트
- 정렬

스토리 조건 판정은 NarrativeService가 담당한다.

## 25.1 발생 흐름

메일이 먼저 현상을 주장해서는 안 된다. 항상 다음 흐름을 따른다.

```text
Behavior System
      ↓
실제 행동 발생
      ↓
Observation Record (eventTick 포함)
      ↓
Narrative Condition
      ↓
Mail 발생
```

`ObservationService`가 실제 행동을 관찰해 `ObservationRecord`를 남기고, `eventTick`에는 관찰 시각이 아니라 실제 simulation tick을 기록한다. Observation 검사는 낮은 주파수로 돌 수 있지만(#37 참고) `eventTick`은 항상 실제 발생 tick이어야 한다.

---

# 26. Narrative 시스템

NarrativeService는 게임 상태를 직접 조작하지 않는다.

역할:

- 조건 확인
- 스토리 플래그 갱신
- 메일 큐 요청
- 특정 연출 이벤트 요청

예:

```ts
type NarrativeFlag =
  | "phase_2_started"
  | "pre_stimulus_behavior_seen"
  | "pattern_21_detected"
  | "unknown_contact_started"
  | "border_detected"
  | "final_protocol_complete";
```

---

# 27. Narrative Condition

조건은 데이터 기반으로 만든다.

예:

```ts
type NarrativeConditionDefinition =
  | { type: "protocolCompleted"; protocolId: ProtocolId }
  | { type: "researchUnlocked"; researchId: ResearchId }
  | { type: "cellCountAtLeast"; count: number }
  | { type: "evidenceRecorded"; evidence: EmergenceEvidenceKind }
  | { type: "observationRecorded"; observation: ObservationKind }
  | { type: "mailReceived"; mailId: MailId }
  | { type: "mailRead"; mailId: MailId }
  | { type: "minDelaySince"; flag: NarrativeFlag; seconds: number }
  | { type: "flag"; flag: NarrativeFlag }
  | { type: "all"; conditions: NarrativeConditionDefinition[] }
  | { type: "any"; conditions: NarrativeConditionDefinition[] };
```

`emergenceAtLeast`(누적 점수 조건)는 사용하지 않는다. Emergence 조건은 항상 Evidence 기반의 `evidenceRecorded`로 표현한다(#28 참고).

새 메일 추가 때문에 NarrativeService 코드를 수정하지 않도록 한다.

## 27.1 스토리 메일 순서

스토리 메일에는 필요에 따라 다음 조건을 둔다.

```text
requiredObservation
requiredPreviousMail
requiredReadState
minimumDelay
```

깨진 데이터 → 문자 → 단어 → 문장 단계가 한꺼번에 쌓이지 않게 한다.

---

# 28. EMERGENCE

Emergence는 숨겨진 값이다.

직접 UI에 표시하지 않는다.

단순한 누적 점수보다 **고유한 행동 증거(Evidence)** 중심으로 관리한다. History(Evidence)와 Current Capability를 분리한다.

## 28.1 Emergence Evidence

```ts
type EmergenceEvidenceKind =
  | "memory"
  | "coordination"
  | "prediction"
  | "autonomy";

interface EmergenceEvidenceRecord {
  kind: EmergenceEvidenceKind;
  firstObservedTick: number;
}
```

Evidence는 종류별로 최초 1회만 기록한다. 같은 행동을 반복해도 무한히 증가하지 않는다. 한 번 획득하면 Trait을 꺼도 사라지지 않고, 체크포인트를 불러올 때만 그 시점의 값으로 되돌아간다(#33 참고). Narrative Phase 해금은 Evidence를 사용한다.

## 28.2 Current Capability

현재 Trait Loadout과 실제 세포 상태에서 계산한다. Evidence가 있어도 현재 관련 능력이 없으면 해당 행동을 화면에서 발생시키지 않는다. 이상 행동을 실제 화면에 발생시키는 조건은 Current Capability다.

EmergenceService는 스토리 트리거용 Evidence/Capability만 계산한다. 세포 능력 그 자체를 여기서 구현하지 않는다.

## 28.3 표기 규칙

가짜 정밀 수치("RANDOM PROBABILITY 0.031%" 등 실제 통계 모델이 없는 수치)를 사용하지 않는다. 측정 해상도보다 과도하게 정밀하게 보이는 문구를 피한다.

---

# 29. Event Bus

이벤트 이름은 문자열 난립을 피한다.

```ts
export interface GameEvents {
  "cell:born": CellBornEvent;
  "cell:divided": CellDividedEvent;
  "cell:died": CellDiedEvent;

  "protocol:preparing": ProtocolPreparingEvent;
  "protocol:started": ProtocolStartedEvent;
  "protocol:completed": ProtocolCompletedEvent;
  "protocol:failed": ProtocolFailedEvent;

  "research:unlocked": ResearchUnlockedEvent;

  "mail:received": MailReceivedEvent;

  "narrative:flagChanged": NarrativeFlagChangedEvent;

  "checkpoint:created": CheckpointCreatedEvent;
  "culture:reseeded": CultureReseededEvent;

  "observation:recorded": ObservationRecordedEvent;
  "discovery:recorded": DiscoveryRecordedEvent;
}
```

기존 이벤트 이름은 임의로 바꾸지 않는다. 새 이벤트만 추가한다.

타입 안전한 EventBus를 사용한다.

---

# 30. 이벤트 사용 규칙

이벤트는 “모든 호출을 이벤트로 바꾸기” 위한 도구가 아니다.

직접 함수 호출이 적절한 경우:

- 같은 feature 내부
- 즉시 결과가 필요한 계산
- 명확한 단일 책임 관계

이벤트가 적절한 경우:

- 다른 feature가 선택적으로 반응
- 발신 시스템이 수신자를 몰라도 됨
- 통계/메일/스토리/오디오처럼 부가 반응

---

# 31. Renderer 구조

Renderer는 읽기 전용 ViewModel을 사용한다.

```ts
interface CultureRenderState {
  cells: readonly CellRenderInfo[];
  threats: readonly ThreatRenderInfo[];
  effects: readonly EffectRenderInfo[];
}
```

Renderer에서 Domain 객체를 직접 변경하지 않는다.

---

# 32. UI Action

UI 입력은 Command 형태로 서비스에 전달한다.

PlayerAction은 두 종류로 나눈다.

- `SimulationCommand`: tick 처리 1단계에서 적용한다. Pause 중에는 거부한다.
- `Transaction`: 시뮬레이션 시간과 무관하게 즉시 검증·확정한다. Pause 중에도 처리할 수 있다(#5.1 참고).

```ts
type SimulationCommand =
  | { type: "feed"; position: Vec2; amount: number }
  | { type: "purge"; position: Vec2; radius: number };
  // signal / isolate / environment 조작은 MVP 이후

type Transaction =
  | { type: "purchaseResearch"; researchId: ResearchId }
  | { type: "setTraitLoadout"; traitIds: CellTraitId[] }
  | { type: "markMailRead"; mailId: MailId }
  | { type: "changeSettings"; settings: SettingsPatch }
  | { type: "saveNow" };

type PlayerAction = SimulationCommand | Transaction;
```

MVP에서 플레이어가 배양 공간에 직접 하는 조작은 `feed`와 `purge`뿐이다. `signal`, `isolate`, 환경 조작 계열 액션은 MVP 이후로 표시하고 타입에 포함하지 않는다.

최종 프로토콜 중 잠기는 조작(Manual Feed, Signal, Purge, Isolate, Environment, Trait Change)은 UI에서만 막지 않는다. Command 처리 서비스가 거부한다.

InputRouter가 이를 적절한 서비스로 전달한다.

---

# 33. Save Data

런타임 객체 전체를 직렬화하지 않는다. 기본 캠페인은 활성 슬롯 1개다.

저장 종류는 Campaign Save, Rolling Autosave(3세대), Protocol Checkpoint, ProfileData로 나눈다. `saveNow`는 슬롯 번호를 받지 않는다.

SaveDataV1에는 simulationTick, resources, research, traits, campaign, inbox, culture, discoveries, observations, emergenceEvidence, rng를 명시적으로 둔다. 저장은 update/Transaction 중간이 아닌 일관된 경계에서 확정한다.

# 34. 저장 범위

세포·위협·환경 Field·Protocol/Preparation Timer·실행된 Action·Cooldown·자원·연구·Trait·메일·Narrative·Discovery·Observation·Emergence·RNG를 저장한다.

Autosave는 초기값 약 30초 simulation time 주기와 Protocol 완료/중요 Transaction/app background 시점에 요청하고 연속 요청은 합친다. 정상본 3세대를 순환하며 최신본이 손상되면 이전 세대로 fallback한다. Failed 상태는 정상 autosave로 덮지 않는다.

SYSTEM → RECOVERY에서는 autosave 세대와 이전 protocol checkpoint를 선택할 수 있다. 과거 checkpoint를 복원하면 이후 campaign/autosave/checkpoint를 폐기한다. ProfileData는 rollback하지 않는다.

Spatial Hash, 파티클, hover, 렌더 보간값은 저장하지 않는다.

# 35. Save Migration

버전은 반드시 가진다.

```ts
switch (data.version) {
  case 1:
    return migrateV1ToLatest(data);
}
```

구 버전 세이브는 별도 migration 함수에서만 변환한다.

기존 SaveData 타입을 무리하게 재사용하지 않는다.

---

# 36. 재현 가능한 난수

디버깅을 위해 SeededRandom을 권장한다.

```ts
interface RandomSource {
  next(): number;
  range(min: number, max: number): number;
}
```

`Gameplay RNG`와 `Presentation RNG`를 분리한다. UI 깜빡임, 노이즈, 파티클 같은 연출용 난수가 게임 결과를 바꾸지 않게 한다. 처음부터 과도하게 여러 RNG를 만들지 않는다. 필요해지면 Gameplay RNG 내부를 Simulation/Protocol/Mutation 같은 substream으로 확장할 수 있으며, 분리 수 자체가 목적은 아니다.

실험 실패를 재현할 수 있어 바이브코딩 디버깅에 도움이 된다.

---

# 37. 성능 목표

Spatial Hash는 기본 도구이지 밀집 상태 성능을 자동으로 보장하는 해결책이 아니다.

## 37.1 MVP 성능 검증

최소 다음 상황을 측정한다.

```text
아군 Cell 300
적 개체 최대 300
Dense Colony
Rapid Bacteria 증식
Signal Field
Toxin Field
CRT 기본 효과
```

## 37.2 장기 목표

`1,500 dynamic entities`는 하드캡이 아니라 성능 시험 기준이다. 실제 생성 한도는 별도 밸런스로 관리한다.

## 37.3 업데이트 주파수 분리

초기 예:

```text
Movement              20 Hz
근거리 감지            20 Hz
장거리 감지             5 Hz
Colony 재구성          2~5 Hz
Narrative Observation   2 Hz
```

실제 값은 프로파일링 후 정한다.

## 37.4 밀집 군집

모든 쌍의 거리를 계산하지 않는다. 가능하면 Spatial bucket, 지역 중심점, 군집 평균, Field 등을 사용한다.

Object Pool, Worker, 병렬 처리 등은 프로파일링 결과가 필요할 때 도입한다.

O(n²) 근접 탐색은 금지한다.

---

# 38. Object Pool

세포 수가 자주 변한다면 객체 생성/삭제 GC가 문제될 수 있다.

필요 시:

- Cell instance pool
- Threat pool
- Particle pool

을 도입한다.

다만 초기 구현부터 모든 것을 풀링하지 않는다.

프로파일링 결과가 있을 때 적용한다.

---

# 39. 핵심 테스트

모든 렌더링을 테스트할 필요는 없다.

게임 규칙을 테스트한다. 다음은 구현 전 필수 테스트 시나리오다.

## 에너지

- 최초 세포가 지속 분열해도 첫 유료 연구에 도달할 수 있다.
- Energy Storage 구매가 ENERGY 생산을 부자연스럽게 막지 않는다.
- 굶주리는 세포가 연구 자원을 계속 생산하지 않는다.
- 저장량이 가득 찬 세포의 넘친 몫이 ENERGY로 들어가지 않는다.
- 내부 에너지가 0인 세포는 Health가 줄고, 에너지를 되찾으면 회복한다.

## Movement

- Chemotaxis 활성 시 영양 gradient 방향으로 평균 이동한다.
- Hazard Avoidance 활성 시 독성 gradient 반대 방향으로 움직인다.
- maxSpeed를 넘지 않는다.
- 경계에 닿은 세포가 Wrap하지 않고 Soft Repulsion/Hard Clamp로 처리된다.

## Research

- prerequisite가 없으면 해금할 수 없다.
- 자원이 부족하면 연구 비용이 차감되지 않는다.
- 해금된 연구를 다시 구매할 수 없다.
- `Running` 중 연구 구매를 시도하면 `PROTOCOL_RUNNING`으로 거부된다.

## Protocol

- Preparing에 들어갈 때 Checkpoint가 정확히 한 번 생긴다.
- 수동 시작과 자동 시작이 같은 전환 함수(`transitionToRunning`)를 거친다.
- 실패하면 Checkpoint를 불러와 Preparing으로 돌아오고 준비 타이머가 처음부터 돈다.
- Checkpoint 이후에 산 연구와 쓴 자원이 함께 되돌아간다(연구만 남고 비용이 돌아오지 않는 문제가 없다).
- 마지막 tick 전멸과 시간 종료가 겹쳐도 실패가 우선된다.
- Population Retention의 필요 개체 수가 시작 개체 수와 무관하다.
- Running이 아닐 때 전멸하면 초기 세포 하나가 다시 생기고 자원과 연구는 유지된다.
- Completed로 넘어가면 남은 위협이 제거된다.
- Idle과 Completed에서 연구 구매와 Trait 변경이 된다.

## 발견/Data

- 같은 발견으로 DATA가 두 번 지급되지 않는다.
- 실패 후 Checkpoint를 불러오면 발견 기록과 DATA가 함께 되돌아가고, 다시 발견하면 다시 지급된다.

## Pause/UI

- Research 화면에서 시간이 멈춘 상태로 구매가 즉시 반영된다.
- Trait 변경도 즉시 반영된다.
- FEED/PURGE는 Pause 중 사용할 수 없다.
- Manual Pause 상태에서 Mail을 닫아도 자동 재개되지 않는다.

## Division

- 첫 분열 이후 부모도 `divisionCooldown`을 따른다.
- 갓 태어난 세포는 `maturityAge`와 `divisionCooldown`을 모두 채워야 분열한다.
- 개체 수 상한 직전에 여러 세포가 동시에 분열을 시작해도 상한을 넘지 않는다.
- `pendingDeath` 세포는 분열하지 않는다.
- 분열 도중 사망 시 자식이 생성되지 않는다.
- 분열 에너지 비용이 복제되지 않는다.

## Trait

- Safe Limit 초과는 허용된다.
- Hard Limit 초과는 거부된다.
- Trait 반복 변경 후 Modifier가 중복 누적되지 않는다.
- 비활성 특성 효과가 남지 않는다.

## Narrative

- 같은 one-shot 메일이 두 번 오지 않는다.
- UNKNOWN 메시지는 선행 조건 없이 발생하지 않는다.
- 최종 메일은 최종 프로토콜 이전에 도착하지 않는다.
- 대사/방어 중심 빌드에서도 대체 Evidence 경로를 통해 진행 가능하다.
- Current Capability가 없는데 History(Evidence)만으로 이상 행동 Mail이 발생하지 않는다.

## Save

- 저장 직후 Load하면 같은 상태를 복원한다.
- 같은 Save에서 같은 입력을 주고 일정 tick 진행 시 게임 결과가 재현된다.
- 이미 실행한 Protocol Action이 Load 후 다시 실행되지 않는다.
- one-shot Mail이 중복 수신되지 않는다.
- V1 → V2 migration 테스트를 추가한다.

## Performance

- Dense Colony 300 Cell + 증식 중인 적 상태를 검증한다.
- CRT 효과를 줄여도 Cell 상태 구분이 유지된다.

---

# 40. 통합 테스트

다음 흐름을 최소 하나씩 자동화한다.

```text
새 게임
→ 세포 생성
→ 먹이 공급
→ 에너지 축적
→ 연구 1개 해금
→ 특성 활성
→ 프로토콜 Preparing 진입 (Checkpoint 생성)
→ 프로토콜 시작
→ 프로토콜 완료
→ DATA 보상
→ 메일 수신
→ 저장
→ 로드
```

```text
프로토콜 Preparing 진입 (Checkpoint 생성)
→ 프로토콜 시작
→ 실패 조건 충족
→ GAME OVER
→ Checkpoint 전체 로드
→ Preparing (타이머 처음부터)
```

이 두 경로가 항상 유지되는 것이 중요하다.

---

# 41. 데이터 검증

게임 시작 시 데이터 파일을 검증한다.

검사 예:

- 존재하지 않는 prerequisite
- 순환 연구 prerequisite
- 존재하지 않는 Trait ID
- 음수 연구 비용
- 중복 ID
- 존재하지 않는 Mail trigger
- Protocol phase 시간 역전

가능하면 Zod 같은 런타임 스키마 검증 도구를 사용한다.

---

# 42. ID 타입

문자열을 아무 곳에서나 직접 쓰지 않는다.

권장:

```ts
type Brand<T, B extends string> = T & { readonly __brand: B };

type CellId = Brand<number, "CellId">;
type ResearchId = Brand<string, "ResearchId">;
type TraitId = Brand<string, "TraitId">;
type ProtocolId = Brand<string, "ProtocolId">;
```

과도하게 복잡해질 경우 최소한 enum/const map이라도 사용한다.

---

# 43. 오류 처리

게임 상태를 깨뜨리는 조용한 실패를 피한다.

예:

```ts
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

Research 구매 실패 이유:

- NOT_ENOUGH_ENERGY
- NOT_ENOUGH_DATA
- PREREQUISITE_MISSING
- ALREADY_UNLOCKED
- PROTOCOL_RUNNING (`Running` 상태에서의 구매 시도, #19.1 참고)

UI는 이 결과를 받아 표시한다.

---

# 44. AI 코딩 규칙

프로젝트 루트에 이미 `AGENTS.md`가 있다면 덮어쓰지 않는다. 기존 한국어 작성 규칙을 유지하면서 아래 개발 규칙을 추가한다.

권장 내용:

```md
# AI Development Rules

- 기존 public interface를 임의로 변경하지 않는다.
- 요청과 관계없는 파일을 수정하지 않는다.
- 새 기능 구현 전에 기존 확장 지점을 찾는다.
- UI에서 게임 상태를 직접 수정하지 않는다.
- Renderer에 게임 규칙을 넣지 않는다.
- balance 값을 코드에 하드코딩하지 않는다.
- any를 사용해 타입 오류를 우회하지 않는다.
- 기존 시스템 전체를 재작성하지 않는다.
- 기존 이벤트 이름과 ID를 임의로 바꾸지 않는다.
- SaveData 변경 시 migration을 추가한다.
- feature 간 직접 의존보다 service contract 또는 event를 우선 검토한다.
- 수정 후 관련 테스트를 실행하고 새 규칙에는 테스트를 추가한다.
```

---

# 45. ARCHITECTURE.md 권장 내용

프로젝트 루트에 별도 문서를 둔다.

반드시 기록할 것:

- 시스템 책임
- 의존 방향
- simulation update 순서
- event 목록
- save version
- data directory 규칙
- 새 feature 추가 절차

AI에게 작업할 때 항상 먼저 읽도록 지시한다.

구현 착수 전에 다음 문서를 함께 준비한다.

```text
AGENTS.md          — AI 작업 규칙
ARCHITECTURE.md     — 시스템 책임, 의존 방향, 이벤트, 저장 구조
CORE_GAME_RULES.md  — 게임 규칙의 정식 원본
```

`CORE_GAME_RULES.md`는 `Dot.exe_두차례_검토_결정사항.md`에서 확정한 게임 규칙을 정식 반영한 문서다. 규칙이 문서마다 다르게 적혀 있을 경우의 우선순위는 머리말을 따른다.

---

# 46. 새 연구 노드 추가 절차

예: 새로운 `Heat Resistance` 연구 추가.

원칙적으로 다음 파일만 수정한다.

1. `data/research/...`
2. 필요하면 Trait 데이터
3. 해당 행동이 완전히 새 기능이라면 관련 system
4. 테스트

ResearchService 자체는 수정하지 않는 것이 정상이다.

---

# 47. 새 적 추가 절차

예: 새로운 Bacteria 변종.

가능하면:

1. `data/threats/`
2. 기존 BacteriaSystem에서 지원되는 파라미터 사용
3. Protocol data 추가
4. 테스트

새로운 행동 메커니즘이 있을 때만 System을 확장한다.

적 하나 추가할 때 ProtocolService, UI, SaveService까지 수정해야 한다면 구조 건강도를 의심해 볼 신호다. 다만 이는 참고 지표이지 강제 규칙은 아니다(#63 참고).

---

# 48. 새 메일 추가 절차

메일 하나 추가 때문에 코드를 수정하지 않는다.

해야 할 일:

1. mail data 작성
2. trigger condition 지정
3. 필요하면 narrative flag data 추가

NarrativeService 내부에 `if (mailId === ...)` 같은 조건문을 계속 추가하지 않는다.

---

# 49. 새 플레이어 액션 추가 절차

예: 새로운 `COOL AREA` 액션.

1. `PlayerAction` 타입 추가
2. 전용 Service 또는 CultureService 확장
3. Input/UI 연결
4. 밸런스 데이터 추가
5. 테스트

UI가 직접 필드를 수정하지 않는다.

---

# 50. 금지 패턴

## 금지 1: God Manager

```text
GameManager.ts
- 세포
- 연구
- 메일
- 세이브
- 프로토콜
- UI
- 사운드
```

이 구조를 만들지 않는다.

## 금지 2: Renderer에서 로직 처리

```ts
if (cell.energy <= 0) {
  cell.dead = true;
}
```

금지.

## 금지 3: UI 직접 상태 변경

금지.

## 금지 4: Boolean 상태 난립

```ts
isDead
isMoving
isHungry
isDividing
isSleeping
```

한 축의 상태는 enum/state machine으로 묶는다.

## 금지 5: Magic Number

```ts
if (cell.energy > 37.5)
```

금지.

## 금지 6: 타입 오류를 any로 덮기

금지.

## 금지 7: 기능 하나 추가하며 대규모 리팩터링

별도 리팩터링 작업으로 분리한다.

---

# 51. State Machine 권장 영역

다음은 상태 머신을 사용한다.

## Cell LifeMode

```text
Active
→ Dividing
→ Active

Active
→ Dormant
→ Active

Active/Dividing/Dormant
→ Dead
```

Health(0~1)와 Infection은 LifeMode와 독립된 축이다. `Damaged`는 LifeMode 값이 아니라 낮은 Health로 표현한다(#8 참고).

## Protocol State

```text
Idle
→ Preparing
→ Running
→ Completed
→ 다음 프로토콜의 Preparing (없으면 Idle)

Running
→ Failed
→ GAME OVER (Pause)
→ Checkpoint 로드
→ Preparing
```

`RetryPreparation` 상태는 두지 않는다(#22.1 참고).

## Narrative Phase

```text
Culture
→ Adaptation
→ Pattern
→ Contact
→ Realization
→ Ending
```

Narrative Phase는 직접 건너뛰지 않는다.

---

# 52. 로그

디버깅을 위해 중요한 상태 변경은 구조화된 로그를 남긴다.

예:

```ts
logger.info("research.unlocked", {
  researchId,
  energySpent,
  dataSpent
});
```

항상 콘솔에 찍는 것이 아니라 DebugLogger를 둔다.

---

# 53. 개발자 디버그 패널

개발 중에는 다음을 볼 수 있게 한다.

- cell count
- simulation tick time
- spatial hash bucket count
- active threats
- field min/max
- current protocol phase
- narrative phase
- emergence evidence 목록 (kind별 최초 기록 여부)
- pause source 목록
- event rate

출시 빌드에서는 숨긴다.

---

# 54. 밸런스 파일 분리

예:

```text
data/balance/
├─ culture.balance.ts
├─ movement.balance.ts
├─ metabolism.balance.ts
├─ protocol.balance.ts
└─ ui.balance.ts
```

시스템 코드와 밸런스 값을 분리한다.

---

# 55. 콘텐츠와 코드 분리 기준

데이터로 해결할 것:

- 연구 비용
- 연구 prerequisite
- 특성 modifier
- 프로토콜 구성
- 메일
- 적 파라미터
- 보상
- 트리거 조건

코드가 필요한 것:

- 새로운 이동 메커니즘
- 새로운 감염 규칙
- 새로운 상태 전이
- 새로운 시스템 상호작용

---

# 56. 버전 관리 단위

작은 작업 단위로 커밋한다.

예:

```text
feat(culture): add chemotaxis steering
test(culture): add chemotaxis direction test
data(research): add chemotaxis research node
ui(research): expose chemotaxis node
```

한 커밋에서 여러 feature를 무작정 섞지 않는다.

---

# 57. 바이브코딩 작업 프롬프트 권장 형식

AI에게 기능을 요청할 때 다음 형식을 권장한다.

```text
목표:
- Chemotaxis 연구를 추가한다.

수정 허용 범위:
- data/research/motility.ts
- features/culture/systems/CellMovementSystem.ts
- 관련 테스트

수정 금지:
- SaveData public schema
- ProtocolService
- MailService
- 기존 event 이름

요구사항:
- 연구 해금 전에는 기존 이동 유지
- 해금 및 활성 후 영양 gradient 방향 steering 추가
- maxSpeed 제한 유지
- 새 규칙에 unit test 추가

작업 후:
- 변경 파일 목록
- 기존 기능 영향
- 테스트 결과
를 보고한다.
```

---

# 58. 완료 기준

기능 하나는 다음 조건을 모두 만족해야 완료로 본다.

- 요구사항 충족
- 타입 오류 없음
- 관련 테스트 통과
- 기존 테스트 통과
- 관련 없는 파일 수정 없음
- balance 값 하드코딩 없음
- SaveData 영향 확인
- event contract 영향 확인
- ARCHITECTURE 문서와 충돌 없음

---

# 59. 초기 개발 순서

## Milestone 1 — Simulation Core

fixed step, Vec2, Cell, movement, boundary, nutrient field, metabolism, starvation/Health, division, death, 216×384 PixelCanvas와 ResponsiveShell 최소 골격을 만든다.

검증 순서는 실제 P-01과 동일하다.

```text
Chemotaxis OFF
→ 가까운 FEED
→ 먹음·분열
→ P-01 완료에 해당하는 지점
→ Chemotaxis 지급
→ 더 먼 FEED
→ 찾아가는 행동 변화
```

Chemotaxis를 처음부터 켠 프로토타입만으로 Milestone 1을 통과할 수 없다.

## Milestone 2 — Research

- resources
- research tree
- trait unlock
- expression load

목표:

플레이어 선택에 따라 세포 행동이 달라진다.

## Milestone 3 — Protocol

- bacteria
- protocol runner
- objectives
- rewards

목표:

첫 번째 생존 실험 완성.

## Milestone 4 — Mail / Narrative

- inbox
- narrative condition
- phase
- system mail

목표:

게임 진행과 메일이 연결된다.

## Milestone 5 — Advanced Threats

- virus
- parasite
- fungus
- environmental threats

## Milestone 6 — Emergence

- collective memory
- prediction
- autonomous signaling
- anomaly behaviors

## Milestone 7 — Final Protocol / Ending

- operator control lock
- autonomous culture
- endings

---

# 60. MVP 범위

첫 구현은 전체 캠페인을 만들지 않는다.

MVP는 다음으로 제한한다.

- 세포 최대 300
- 자원 3종: ENERGY, DATA, NUTRIENT RESERVE
- Field 3종: nutrient, toxin, signal(Alarm Signal 전용)
- 기본 이동, 배양 공간 경계
- 분열
- 전역 Trait Loadout
- 플레이어 조작: FEED, PURGE (SIGNAL/ISOLATE/환경 조작은 MVP 이후)
- 실제 효과가 연결된 연구 10개: Enhanced Glycolysis, Energy Storage, Accelerated Mitosis, Density Sensing, Chemotaxis, Hazard Avoidance, Reinforced Membrane, Phagocytosis, Alarm Signal, Adhesion
- Rapid Bacteria
- 프로토콜 3개: P-01 CULTURE EXPANSION, P-02 RAPID BACTERIA, P-03 LIMITED NUTRIENT
- 메일 8~10개
- Narrative Phase 1~2. Phase 2 메일은 감염 실험 안내와 방어 연구에 대한 반응으로 한정한다(사전 반응 메일 제외)
- 한국어·영어 현지화. 화면의 글은 처음부터 `Game/Locale/*.json`의 키로 가져온다(#19.2)

전역 환경값(Temperature/pH/Oxygen), 폐기물/CONTAMINATION, Cluster 생성과 colonyId, 플레이어 SIGNAL은 MVP에서 제외한다.

MVP에서 재미를 먼저 검증한다.

검증 질문:

1. 점이 움직이는 것만 봐도 살아 있다는 느낌이 드는가?
2. 먹이 위치를 바꾸는 행동이 재미있는가?
3. 다음 프로토콜을 준비하는 선택이 실제 고민을 만드는가?
4. 연구가 화면의 행동을 눈에 띄게 바꾸는가?
5. 세포가 100개 이상일 때 시각적으로 풍부한가?

---

# 61. 첫 프로토타입에서 반드시 확인할 것

1. Chemotaxis가 없어도 초반 5~10분의 관찰·FEED·분열이 지루하지 않은가?
2. Chemotaxis 획득 직후 같은 조작의 의미가 분명히 달라져 성장 체감이 생기는가?

둘 중 하나라도 실패하면 P-01 또는 기본 이동/먹이 상호작용을 먼저 고친다.

# 62. 기술적 충돌 점검

## 62.1 연속 이동과 Spatial Grid

문제 없음.

Spatial Grid는 근접 검색 최적화용이며 이동 타일이 아니다.

## 62.2 여러 System이 Cell을 동시에 수정

위험 요소다.

해결:

- 실행 순서 고정
- 각 시스템이 소유하는 필드 범위 명확화
- 필요하면 Command/Delta를 모아 마지막에 적용

## 62.3 Narrative가 게임 로직을 강제로 바꾸는 문제

NarrativeService가 Cell을 직접 수정하지 않는다.

특수 연출이 필요하면 명시적 Event 또는 Protocol Action으로 요청한다.

## 62.4 모든 Trait을 데이터화하다 복잡해지는 문제

수치 Modifier는 데이터화한다.

완전히 새로운 행동은 코드 구현을 허용한다.

“모든 것을 데이터로 해결”하지 않는다.

## 62.5 EventBus 남용

즉시 반환값이 필요한 내부 계산은 직접 호출한다.

Cross-feature notification에만 EventBus를 우선한다.

---

# 63. 최종 아키텍처 원칙

이 프로젝트에서 가장 중요한 질문은 다음이다.

> 새 기능 하나를 추가할 때 몇 개의 기존 시스템을 직접 수정해야 하는가?

콘텐츠 하나(새 연구 데이터, 새 Mail, 기존 행동을 사용하는 새 Threat 변형, 새 Protocol 데이터) 추가 시 기존 시스템 **0~2개 수정**은 강제 규칙이 아니다. 구조가 건강한지 판단하는 지표로만 사용한다.

새 연구 노드, 메일, 프로토콜, 적 파라미터처럼 콘텐츠 성격의 변경은 가능한 한 데이터 추가만으로 끝나야 한다.

새로운 근본 메커니즘일 때는 Domain, Save, UI, Test, System이 함께 바뀌는 것이 정상일 수 있다. 새로운 근본 메커니즘일 때만 해당 feature의 System을 확장하며, 이런 정상적인 구조 변경까지 실패로 취급하지 않는다.

이 구조를 유지하면 바이브코딩으로 기능을 계속 추가해도 프로젝트 전체를 다시 쓰게 되는 위험을 크게 줄일 수 있다.
