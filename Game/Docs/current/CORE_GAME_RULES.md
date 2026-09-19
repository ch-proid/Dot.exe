# Dot.exe CORE GAME RULES

> 문서 버전: 1.4 (2026-09-19)  
> 이 문서가 게임 규칙의 **정식 원본**이다. 기획서와 명세서의 `결정 N절`, `정본 N절`은 이 문서의 N절을 가리킨다.  
> 규칙을 바꿀 때는 이 문서를 먼저 고치고 문서 끝 `변경 기록`에 적은 뒤 기획서·명세서·코드를 맞춘다.  
> 아직 플레이테스트를 거치지 않은 수치는 `초기 밸런스 값`이며 고정 규칙이 아니다. 실제 값은 `Game/src/data/balance/`에 둔다.  
> 출처: 세 차례 검토의 결정(`Game/Docs/history/2026-09-19_DECISION_LOG_V1.2.md`, v1.2에서 동결).

---

# 1. 문서 우선순위

게임 규칙이 여러 문서에서 다르게 적혀 있을 경우 다음 순서로 해석한다.

1. `Game/Docs/current/CORE_GAME_RULES.md`
2. `Game/Docs/current/DEVELOPMENT_SPEC.md`
3. `Game/Docs/current/GAME_DESIGN.md`
4. 밸런스 데이터
5. 예시 문구 및 연출용 수치

`Game/Docs/history/`는 검토 과정과 결정 이력을 보존하는 기록이며 구현 기준이 아니다. `.agents/`는 Luna Chat Coder 통합 전용이며 게임 규칙·기획·코드를 두지 않는다.

규칙을 바꿀 때는 이 문서를 먼저 수정하고 다른 현행 문서와 코드를 맞춘다.

# 2. 명칭

게임 작품명과 게임 안에서 연구원이 실행하는 시스템의 이름을 모두 **`Dot.exe`**로 통일한다.

- 작품명: `Dot.exe`
- 극중 프로그램/배양 시스템명: `Dot.exe`
- 타이틀, 부팅 화면, 시스템 헤더도 `Dot.exe`를 사용한다.

별도의 시스템명은 사용하지 않는다. `CULTURE`, `RESEARCH`, `MAIL` 등은 기능·화면 이름이므로 그대로 사용할 수 있다.

# 3. 발현 부하(Expression Load)

## 3.1 기본 규칙

발현 부하는 두 단계 한도로 관리한다.

- `Safe Expression Limit`
- `Hard Expression Limit`

Safe Limit까지는 정상 작동한다.

Safe Limit를 초과하면 특성 활성은 가능하지만 과발현 패널티가 발생한다.

Hard Limit를 초과하는 조합은 활성화할 수 없다.

## 3.2 초기 밸런스 값

다음 수치는 **초기안**이며 플레이테스트 후 조정할 수 있다.

```text
Safe Limit = 100
Hard Limit = 125
```

예시:

```text
0 ~ 100      정상
100 초과     과발현 상태
125 초과     활성 불가
```

## 3.3 과발현 패널티

과발현 상태에서는 다음 계열의 비용이 증가한다.

- 기초 대사 부담
- 스트레스 발생
- 분열 비용
- 유지 효율 저하

정확한 계수는 밸런스 데이터에서 관리한다.

## 3.4 구현 규칙

`TraitLoadoutService`는 Safe Limit 초과를 거부하지 않는다.

Hard Limit 초과만 거부한다.

---

# 4. 세포 에너지와 SYSTEM ENERGY

기존의 “내부 저장 한도를 넘은 에너지만 연구 자원으로 회수”하는 방식은 사용하지 않는다.

분열과 연구 자원이 서로 막히는 문제를 피하기 위해 **대사 순생산을 분배하는 방식**을 사용한다.

## 4.1 에너지 흐름

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
Cell Internal Energy  SYSTEM ENERGY
```

## 4.2 별도 관리 값

다음 값은 서로 다른 개념이다.

```text
Survival Reserve
Division Requirement
Maximum Storage
System Harvest Ratio
```

서로 같은 값으로 사용하지 않는다.

## 4.3 생존 우선 규칙

세포 내부 에너지가 `Survival Reserve`보다 낮으면 SYSTEM ENERGY 회수를 중단한다.

```text
Cell Energy < Survival Reserve
→ 순생산 100%를 세포 내부에 배정

Cell Energy >= Survival Reserve
→ 순생산 일부를 내부 저장, 일부를 SYSTEM ENERGY로 배정
```

## 4.4 Energy Storage 연구

`Energy Storage`는 `Maximum Storage`만 증가시킨다.

SYSTEM ENERGY 회수 기준이나 회수 비율을 직접 올리지 않는다.

## 4.5 분열 에너지 보존

분열 시 에너지가 새로 생성되지 않는다.

```text
부모 내부 에너지
- 분열 비용
↓
남은 에너지
↓
부모와 자식에게 분배
```

자식 세포가 별도의 무료 에너지를 받고 태어나지 않는다.

남은 에너지는 부모와 자식이 반씩 나눈다.

## 4.6 경계 경우

- 내부 에너지가 `Maximum Storage`에 찬 세포도 섭취와 대사를 계속한다.
- 저장하지 못한 내부 몫은 버린다. SYSTEM ENERGY로 돌리지 않는다. (돌리면 “넘친 만큼 회수” 방식이 되살아나 Energy Storage 구매가 SYSTEM ENERGY 생산을 깎는다.)
- 순생산이 음수면 내부 에너지에서 뺀다.
- 특성 변경으로 `Maximum Storage`가 줄면 넘는 몫은 바로 사라진다.

## 4.7 굶주림

내부 에너지가 0이면 Health가 밸런스 데이터의 속도로 줄어든다. Health가 0이 되면 `PendingDeath`가 된다.

에너지를 되찾으면 Health는 천천히 회복한다.

---

# 5. 프로토콜 목표와 생존 판정

범용 `SURVIVAL RATE` 하나로 모든 실험을 판정하지 않는다.

프로토콜 목표를 명시적인 목적 단위로 나눈다.

예:

- `Survive Duration`
- `Population Retention`
- `End Population`
- `Avoid Extinction`
- `Maintain Infection Below`
- `Maintain Environment`
- `Observe Autonomous Response`

## 5.1 Viable Cell 정의

기본적인 생존 가능 개체 판정은 다음 기준을 사용한다.

포함:

- `LifeMode.Active`
- `LifeMode.Dividing`
- `LifeMode.Dormant`
- Health가 0보다 큰 개체

제외:

- `PendingDeath`
- `LifeMode.Dead`
- 배양 공간에서 완전히 제거된 분석 샘플

`Damaged`는 별도 LifeMode가 아니라 Health 상태로 취급한다.

## 5.2 Population Retention

프로토콜에는 필요하면 다음 값을 둔다.

```text
referencePopulation
retentionRatio
absoluteMinimum
evaluationWindow
```

판정 기준:

```text
requiredPopulation =
max(
  absoluteMinimum,
  ceil(referencePopulation * retentionRatio)
)
```

기준은 프로토콜이 정한 `referencePopulation`만 쓴다. 시작 개체 수는 기준에 넣지 않는다.

적은 개체 수로 시작해 기준을 낮출 수도 없고, 많이 키워 들어간 플레이어가 더 높은 기준을 떠안지도 않는다.

HUD에는 비율이 아니라 필요 개체 수를 보여준다.

```text
REQUIRED 060
```

## 5.3 마지막 구간 유지

“마지막 30초 유지” 같은 조건은 평균이 아니다.

해당 구간의 **모든 판정 tick에서 조건을 충족**해야 한다.

## 5.4 다중 목표

기본적으로 필수 목표는 `AND` 관계다.

보너스 목표만 선택 조건으로 둔다.

## 5.5 마지막 tick 우선순위

마지막 tick에 시간 종료와 전멸이 동시에 발생하면:

```text
실패 조건 판정
→ 성공 조건 판정
→ 보상 지급
```

순서로 처리한다.

전멸 상태에서 시간만 끝났다고 성공하지 않는다.

---

# 6. 프로토콜 상태와 체크포인트

프로토콜은 설명을 읽는 시간과 실제 준비 압박을 분리한다. **Briefing은 무제한**, **Preparing은 제한시간**이다.

## 6.1 상태 흐름

```text
Idle
→ Briefing
→ Preparing
→ Running
→ Completed
→ 다음 프로토콜의 Briefing (없으면 Idle)

Running
→ Failed
→ GAME OVER
→ 체크포인트 로드
→ Preparing
```

- `Briefing`: simulation pause. 목표·예상 위협·보상을 읽는다. 연구 구매·Trait 변경·FEED/PURGE는 할 수 없다.
- `Preparing`: simulation과 Preparation Timer가 흐르는 live-planning 구간. 연구·Trait·FEED/PURGE·증식·자원 생산을 제한시간 안에서 처리한다.
- `Running`: 연구와 Trait 변경을 잠근다.
- `RetryPreparation`은 두지 않는다.

## 6.2 체크포인트 생성 시점

`Briefing → Preparing` 전환에서 Preparing에 들어가는 순간 정확히 한 번 생성한다. 체크포인트를 먼저 확정한 뒤 준비 타이머를 시작한다.

이전 프로토콜 체크포인트도 캠페인 동안 보존한다. SYSTEM → RECOVERY에서 더 앞의 체크포인트를 선택할 수 있다.

## 6.3 Running 전환

수동 시작과 자동 시작 모두 `transitionToRunning()`을 사용한다. Preparation Timer가 0이면 자동 시작한다. 남은 시간은 모든 준비 UI에서 항상 보인다.

## 6.4 실패 후 복구

실패하면 GAME OVER 뒤 체크포인트를 통째로 불러오고 Briefing을 다시 강제하지 않은 채 Preparing으로 돌아간다. 준비 타이머는 처음부터 다시 돈다.

# 7. 복원 범위와 연구 구매

## 7.1 복원 범위

체크포인트를 불러오면 세포·적·환경 Field, 자원, 연구, Trait Loadout, Protocol 상태, 메일, Narrative Flag, Observation, 발견 기록, Emergence Evidence, simulationTick, RNG를 모두 그 시점으로 되돌린다.

체크포인트와 무관하게 유지하는 것은 ProfileData의 엔딩 기록과 모드 해금 정보뿐이다.

## 7.2 연구 구매와 Trait 변경

- `Briefing`: 조회만 가능. 구매·변경 금지.
- `Preparing`: 허용. 제한시간 안의 핵심 선택 구간.
- `Running`: 금지.
- `Idle`, `Completed`: 허용.

## 7.3 Running이 아닐 때의 전멸

Running이 아닐 때 Viable Cell이 0이 되면 게임 오버로 처리하지 않는다. 초기 세포 하나를 중앙에 다시 만들고 자원과 연구는 유지한다.

## 7.4 프로토콜 완료 시

Completed로 넘어갈 때 남아 있는 위협을 제거한다.

# 8. 발견 기록과 DATA 보상

발견은 종류별로 한 번만 기록하고, 기록하는 순간 DATA를 지급한다.

```ts
DiscoveryRecord {
  observed: boolean;
}
```

미확정(Pending) DATA는 두지 않는다.

실패해서 체크포인트를 불러오면 발견 기록과 DATA가 함께 되돌아가므로, 다시 하면서 다시 발견하고 다시 받는다.

- 실패 후 영구히 보상을 잃는 문제
- 재도전마다 무한 DATA를 받는 문제

둘 다 생기지 않는다.

---

# 9. 게임 시간과 UI 명령 처리

Simulation Time과 UI/Transaction Processing을 분리한다. **Preparing은 시간 압박을 위한 live-planning 예외**다.

## 9.1 화면과 시간

- `Briefing`: simulation pause, 준비 타이머 시작 전.
- `Preparing`: simulation과 Preparation Timer 진행. RESEARCH, Trait Loadout, MAIL, ANALYSIS, SYSTEM을 열어도 멈추지 않으며 남은 시간을 항상 표시한다. 일반 Manual Pause는 사용할 수 없다.
- `Preparing` 이외: CULTURE 외 화면은 기존처럼 simulation pause.
- 앱 background/OS interruption은 `SYSTEM_SUSPEND`로 모든 시간을 멈춘다. 복귀 시 밀린 시간을 소급하지 않는다.

## 9.2 Pause 중 즉시 처리 가능한 명령

Preparing 이외의 pause 상태에서는 상태 규칙이 허용하는 연구 구매, Trait 변경, 메일 읽음, 설정 변경, 저장을 Transaction으로 처리할 수 있다.

## 9.3 Pause 중 금지되는 시뮬레이션 명령

FEED, SIGNAL, PURGE, 환경 조작, 전투성 개입은 pause 상태에서 사용할 수 없다.

## 9.4 Pause Source

`MANUAL_PAUSE`, `BRIEFING`, `MAIL_MODAL`, `RESEARCH_MODAL`, `ANALYSIS_MODAL`, `SYSTEM_MODAL`, `GAME_OVER`, `SYSTEM_SUSPEND`를 구분한다. Preparing에서는 modal source를 추가하지 않으며 SYSTEM_SUSPEND만 준비 시간을 멈춘다.

## 9.5 오프라인 진행

기본 캠페인에는 오프라인 진행이 없다.

# 10. Tick 처리 순서

고정 스텝 기반 시뮬레이션을 사용한다.

초기 권장:

```text
Simulation: 20 Hz
Render: requestAnimationFrame
```

정확한 주파수는 성능 검증 후 조정할 수 있다.

## 10.1 한 Tick의 처리 순서

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

## 10.2 중요한 처리 원칙

- 치명 피해를 받은 세포는 같은 tick에 분열하지 못한다.
- 신규 세포는 다음 tick부터 행동한다.
- Protocol 성공/실패 판정은 해당 tick의 피해·사망 결과가 확정된 뒤 처리한다.
- 여러 세포가 같은 영양분을 먹으려 할 때 배열 순서로 선착순 독점하지 않는다.
- 경쟁 자원은 요청량을 모아 동시에 분배한다. 요청이 공급을 넘으면 요청량에 비례해 나눈다.
- 위협 개체도 같은 순서를 따른다. 감지·이동은 5~8단계, 공격·섭취는 10~12단계, 증식과 사망은 16~18단계에서 세포와 함께 처리한다.

---

# 11. 세포 상태 모델

기존의 `Active / Dividing / Dormant / Damaged / Dead` 단일 enum은 사용하지 않는다.

## 11.1 LifeMode

```text
Active
Dividing
Dormant
Dead
```

## 11.2 Health

```text
0.0 ~ 1.0
```

손상 정도는 Health로 표현한다.

## 11.3 Infection

감염은 독립 상태다.

```text
InfectionState?
```

따라서 다음 조합이 가능하다.

```text
Dormant + Low Health + Infected
Dividing + Damaged
Active + Infected
```

Health를 별도 축으로 둔 이유는 LifeMode와 손상 상태가 동시에 존재할 수 있기 때문이다.

---

# 12. 분열 규칙

세포 생애 나이와 재분열 준비 시간을 분리한다.

세포에는 최소한 다음 값이 필요하다.

```text
age
lastDivisionTick
divisionCooldown
divisionDuration
divisionProgress
```

## 12.1 분열 시간 조건

첫 분열과 그 뒤의 분열을 가르지 않고 항상 두 조건을 함께 본다.

```text
age >= maturityAge
AND
currentTick - lastDivisionTick >= divisionCooldown
```

최초 세포의 `lastDivisionTick`은 생성된 tick이다.

`lastDivisionTick`은 분열이 완료된 시점에 기록한다.

## 12.2 기본 조건과 개체 수 상한

기본 분열 조건은 내부 에너지, 시간 조건, 스트레스, 개체 수 상한이다.

지역 밀도는 기본 조건에 넣지 않는다. 고밀도에서 분열을 억제하는 것은 Density Sensing 연구의 효과다. 연구 없이 과밀해진 군집은 영양 경쟁으로 대가를 치른다.

개체 수 상한은 분열을 시작할 때 `Viable Cell 수 + 분열 중인 세포 수`로 검사한다. 비용만 내고 자식이 생기지 않는 일이 없게 한다.

## 12.3 Accelerated Mitosis

MVP에서는 다음 효과를 가진다.

- `divisionCooldown` 감소
- `divisionDuration` 감소
- 분열 Energy 비용 증가

기존의 “돌연변이 증가” 대가는 MVP에서 사용하지 않는다.

Mutation 시스템은 후속 버전으로 미룬다.

## 12.4 분열 비용

분열 시작 시 Energy를 예약·소모한다.

분열 도중 사망하면 반환하지 않는다.

성공 시 남은 내부 에너지를 부모와 자식에게 나눈다.

```text
parent.age = 유지
child.age = 0

parent.lastDivisionTick = now
child.lastDivisionTick = now
```

감염 상속 규칙은 감염 종류별로 따로 정의한다.

---

# 13. Trait 적용 범위

MVP에서는 **배양체 전체가 하나의 전역 Trait Loadout을 공유**한다.

`Cell.activeTraits`를 개별 기준으로 사용하지 않는다.

구조:

```text
CULTURE GENOME
      ↓
Trait Loadout
      ↓
모든 일반 세포
```

개별 차이는 이후 다음 구조로 추가한다.

- mutation
- role
- phenotypeModifier

## 13.1 Trait 변경 가능 시점

금지:

- Running

그 밖의 상태(Idle, Preparing, Completed)에서는 허용한다.

즉 실험 도중 상황을 보고 생산 특성을 방어 특성으로 즉시 갈아끼우는 플레이를 막는다.

---

# 14. MVP 연구 목록

MVP 연구는 단순 이름이 아니라 실제 검증 가능한 효과를 가진다.

## METABOLISM

### Enhanced Glycolysis
- 대사 생산 증가
- Nutrient 소비 증가

### Energy Storage
- Cell Maximum Storage 증가
- MVP에서 대가는 발현 부하뿐이다. 기획서의 “분열 속도 감소”는 쓰지 않는다.

## REPRODUCTION

### Accelerated Mitosis
- 재분열 준비 시간 감소
- 분열 소요 시간 감소
- 분열 Energy 비용 증가

### Density Sensing
- 고밀도에서 분열 억제

## MOTILITY

### Chemotaxis
- Nutrient gradient 추적
- 구매하지 않는다. P-01 완료 보상으로 연구 완료 상태로 지급하고 바로 활성화한다.

### Hazard Avoidance
- Toxin gradient 반대 방향으로 이동

## DEFENSE

### Reinforced Membrane
- Rapid Bacteria 접촉 피해 감소

### Phagocytosis
- 조건을 만족한 세포가 작은 Bacteria를 접촉 제거

### Alarm Signal
MVP에서는 한 노드에 최소 행동을 묶는다.

1. 적을 감지한 세포가 Threat Signal을 발생
2. 주변 세포가 해당 신호를 감지
3. Phagocytosis가 활성화되어 있으면 신호 근원 쪽으로 접근
4. 그렇지 않으면 위험 방향에서 약하게 이탈

## COLONY

### Adhesion
- 인접 세포 사이 결속력 발생
- MVP에서는 끌어당기는 힘만 구현한다. 군집 ID와 Cluster 생성(18절)은 MVP 이후다.

---

# 15. MVP Protocol

## P-01 CULTURE EXPANSION

초기에는 Chemotaxis가 없다. 플레이어는 세포 가까이에 영양분을 공급하고 기본 불규칙 운동·섭취·분열로 첫 성장을 만든다.

P-01 완료 보상으로 Chemotaxis를 연구 완료 상태로 지급하고 즉시 활성화한다. 이후 같은 FEED를 더 먼 곳에 사용했을 때 세포가 스스로 찾아가는 변화가 첫 번째 큰 성장 보상이 된다.

첫 프로토타입도 본편과 같은 순서를 따른다.

```text
Chemotaxis OFF
→ 가까운 FEED로 성장·분열
→ P-01 완료에 해당하는 지점
→ Chemotaxis 지급/활성
→ 더 먼 FEED
→ 군집 흐름 변화 확인
```

Chemotaxis를 처음부터 켜 둔 프로토타입만으로 핵심 재미를 검증했다고 보지 않는다.

## P-02 RAPID BACTERIA

Rapid Bacteria 최초 투입.

기본 PURGE만으로도 어렵게 통과할 수 있다.

다음 연구가 있으면 크게 유리하다.

- Reinforced Membrane
- Phagocytosis
- Alarm Signal

정답 Trait 하나를 강제하지 않는다.

Rapid Bacteria는 MVP에서 약한 Toxin도 생성한다.

따라서 Hazard Avoidance를 실제로 검증할 수 있다.

## P-03 LIMITED NUTRIENT

- Rapid Bacteria
- 제한된 Nutrient
- 자원 경쟁

단순 공격력보다 자원 관리와 이동이 중요해진다.

“제한”은 NUTRIENT RESERVE의 보충 속도를 줄이는 것이다. Rapid Bacteria는 Nutrient를 먹어야 증식하므로 세포와 같은 영양을 두고 경쟁한다.

## MVP의 플레이어 조작

MVP에서 플레이어가 배양 공간에 하는 조작은 FEED와 PURGE다.

SIGNAL은 받을 세포가 없으므로(Signal Detection이 MVP 연구에 없다) 넣지 않는다. Signal Field는 Alarm Signal 전용으로 쓴다.

---

# 16. MVP 자원

MVP에도 다음 세 자원을 포함한다.

```text
SYSTEM ENERGY
DATA
NUTRIENT RESERVE
```

DATA는 MVP에서는 자동 분석 방식만 사용한다.

획득 후보:

- 새 위협 최초 관찰
- 새 행동 최초 관찰
- Protocol 성공

관찰하거나 성공하는 순간 바로 지급한다. (8절)

능동적인 Analysis 미니 시스템은 MVP 이후로 미룬다.

---

# 17. 환경 규칙

## 17.1 배양 공간 경계

배양 공간은 닫힌 사각형 Chamber다.

반대편으로 Wrap하지 않는다.

경계 처리:

```text
Soft Repulsion
+
Hard Clamp
+
Boundary Normal 방향 속도 감소
```

후반 “경계 발견” 서사와 양립해야 한다.

## 17.2 MVP Field

공간 Field:

- Nutrient
- Toxin
- Signal (Alarm Signal 전용)

전역 환경값(Temperature, pH, Oxygen), FLOW, 폐기물과 CONTAMINATION은 MVP에서 제외한다. P-01~03 어디에서도 게임 결과를 바꾸지 않기 때문이다. ENVIRONMENT 조작도 MVP에 없다.

## 17.3 Oxygen (MVP 이후)

처음 넣을 때는 공간 Oxygen Field를 만들지 않는다.

Global Oxygen 하나를 사용한다.

세포 전체 소비량에 따라 감소하고 환경 장치가 보충한다. 부족할 때 세포가 받는 효과는 넣는 시점에 정한다.

밀집 군집 내부에서만 저산소가 되는 지역 Oxygen은 그 뒤로 미룬다.

## 17.4 ISOLATE

MVP에서는 제외한다.

이 기능은 다음과 동시에 설계해야 한다.

- 이동 차단
- 공격/감염 통과
- 신호 통과
- Nutrient/Toxin 확산
- 비용
- 지속 시간
- 영역 중첩

전체 캠페인 구현 전 별도로 확정한다.

---

# 18. 군집 생성

MVP 이후 기능이다. MVP의 Adhesion은 결속력만 구현한다.

부착의 결속력은 아직 군집이 없는 세포끼리도 작용한다. 같은 군집 세포만 끌어당기면 첫 군집이 만들어지지 않는다.

세포는 처음부터 모두 `colonyId`를 갖지 않는다.

Adhesion이 활성화된 상태에서 일정 시간 가까이 유지된 세포들이 Cluster를 형성한다.

```text
접근
→ 접촉 유지
→ Cluster 생성
```

군집끼리 연결되면 병합할 수 있다.

충분히 오래 떨어져 있으면 분리할 수 있다.

Colony 재구성은 매 simulation tick마다 실행할 필요가 없으며 낮은 주파수로 갱신할 수 있다.

---

# 19. Emergence 시스템

단순한 누적 점수보다 **고유한 행동 증거(Evidence)** 중심으로 관리한다.

예:

```text
MEMORY EVIDENCE
- 동일 위협 재노출 시 반응 변화

COORDINATION EVIDENCE
- 다수 세포가 동일 위협에 집단 대응

PREDICTION EVIDENCE
- 자극 이전 행동 변화

AUTONOMY EVIDENCE
- 외부 Signal 없이 집단 행동 개시
```

## 19.1 History와 Current Capability 분리

### Emergence History / Evidence

- 한 번 획득하면 특성을 꺼도 사라지지 않는다. 체크포인트를 불러올 때만 그 시점의 값으로 돌아간다.
- Narrative Phase 해금에 사용한다.
- 같은 행동 반복으로 무한히 증가하지 않는다.

### Current Capability

- 현재 Trait Loadout과 실제 세포 상태가 결정한다.
- 이상 행동을 실제 화면에서 발생시키는 조건이다.

Evidence가 있어도 현재 관련 능력이 없으면 해당 행동을 발생시키지 않는다.

## 19.2 캠페인 경로

한 연구 트리만 강제하지 않는다.

여러 계통에서 동일한 고차 행동 Evidence를 얻을 수 있게 한다.

예:

Memory 계열:

- Adaptive Immunity
- Response Conditioning
- Persistent Signal

Coordination 계열:

- Collective Defense
- Resource Sharing
- Collective Response

## 19.3 캠페인 요구 방식

특정 노드 하나를 강제하기보다 행동 계열을 요구한다.

예:

```text
다음 중 하나의 고등 집단 행동을 관찰:

- Collective Defense
- Resource Sharing
- Signal Relay
```

---

# 20. 이상 행동과 메일

메일이 먼저 현상을 주장해서는 안 된다.

항상 다음 흐름을 따른다.

```text
Behavior System
      ↓
실제 행동 발생
      ↓
Observation Record
      ↓
Narrative Condition
      ↓
Mail 발생
```

## 20.1 사전 반응 예

```text
세포가 실제로 자극 전에 움직임
↓
Observation:
PRE_STIMULUS_RESPONSE
eventTick 기록
↓
Narrative 조건 충족
↓
J. Park 메일
```

## 20.2 경계 발견

평소 벽에 부딪히는 행동만으로는 “경계 발견”으로 처리하지 않는다.

후반에는 다음과 같은 명확한 행동이 필요하다.

- 여러 구간의 경계를 반복 탐색
- 경계선을 따라 배열
- 경계 주변 신호 교환
- 일정 시간 지속

이 행동이 실제 화면에서 발생한 뒤:

```text
WE FOUND THE BORDER.
```

메일이 온다.

## 20.3 Mail 순서

스토리 메일에는 필요에 따라 다음 조건을 둔다.

```text
requiredObservation
requiredPreviousMail
requiredReadState
minimumDelay
```

깨진 데이터 → 문자 → 단어 → 문장 단계가 한꺼번에 쌓이지 않게 한다.

---

# 21. Observation 시간 해상도

Simulation은 예를 들어 20Hz로 동작하고 Narrative Observation은 더 낮은 빈도로 검사할 수 있다.

다만 사건 발생 시각은 실제 simulation tick에 기록한다.

```text
Simulation Event 발생
→ eventTick 저장
→ ObservationSystem이 이후 확인
```

ObservationSystem이 2Hz라고 해서 사건 시각 자체가 0.5초 단위가 되는 것은 아니다.

UI 문구는 측정 해상도보다 과도하게 정밀하게 보이지 않도록 한다.

권장:

```text
RESPONSE PRECEDES STIMULUS
BY APPROX. 1.8 SEC
```

실제 통계 모델이 없다면:

```text
RANDOM PROBABILITY 0.031%
```

같은 가짜 정밀 수치는 사용하지 않는다.

대신:

```text
REPEATED STRUCTURE DETECTED

EXPECTED RANDOM MATCH:
UNLIKELY
```

같은 표현을 사용한다.

---

# 22. 최종 프로토콜

`WITHOUT OPERATOR SUPPORT`는 에너지원이 없는 상태를 뜻하지 않는다.

정확한 뜻은:

> **운용자의 수동 개입 없이 배양체가 스스로 유지·대응하는지 검증한다.**

## 22.1 최종 프로토콜 중 금지되는 조작

```text
Manual Feed       LOCKED
Signal            LOCKED
Purge             LOCKED
Isolate           LOCKED
Environment       LOCKED
Trait Change      LOCKED
```

잠금은 UI에만 적용하지 않는다.

Command 처리 서비스에서도 거부한다.

## 22.2 시험 단계

초기안:

### Phase A — EQUILIBRIUM
약 90초.

기초 Nutrient 유입만 존재.

세포가 스스로 안정적인 상태를 만드는지 본다.

### Phase B — DISTURBANCE
약 60초.

예:

- 병원체
- Oxygen 감소
- Toxin 증가
- Nutrient 유입 감소

### Phase C — RECOVERY
약 90초.

외부 충격 이후 군집이 스스로 안정 상태로 돌아오는지 본다.

시간 수치는 플레이테스트 전 초기안이다.

## 22.3 성공 조건

단순 생존만으로 성공하지 않는다.

예:

```text
전멸하지 않음
AND
최종 인구 조건 충족
AND
폐기물/오염 임계값 이하
AND
Autonomous Response 최소 1회 관찰
AND
Recovery 구간에서 회복 추세 확인
```

Autonomous Response는 빌드마다 달라도 된다.

예:

- 자동 감염 격리
- 자율 방어 집결
- 자동 휴면 전환
- 자원 재분배
- 자동 경보

특정 연구 하나만 강제하지 않는다.

## 22.4 진입 자원

최종 실험 시작 시 외부 배양 환경은 표준 조건으로 초기화할 수 있다.

예:

- Nutrient Field
- Toxin Field
- Signal Field
- Environment

Cell 내부 Energy는 유지한다.

저장 빌드도 유효한 전략으로 인정하되, 단순 비축만으로 모든 성공 조건을 만족할 수 없게 한다.

## 22.5 엔딩 판정 문구

`LONG-TERM STABILITY`처럼 실제 시험 범위를 넘는 표현을 피한다.

권장:

```text
AUTONOMOUS STABILITY TEST
PASS
```

---

# 23. 저장 규칙

저장은 일관된 state boundary에서 수행하고 쓰기 실패 시 직전 정상본을 보존한다. MVP/첫 출시에서는 복잡한 슬롯 관리보다 모바일의 CONTINUE 경험을 우선한다.

## 23.1 저장 계층

1. **Campaign Save 1개** — 타이틀의 CONTINUE 대상. NEW GAME은 덮어쓰기 확인을 받는다.
2. **Rolling Autosave 3세대** — Campaign Save를 덮기 전 최근 정상본을 순환 보관한다. 최신본 손상 시 이전 세대로 fallback한다.
3. **Protocol Checkpoint** — Preparing 진입 시 생성하는 별도 복구 지점. 일반 저장과 슬롯을 공유하지 않는다.
4. **ProfileData** — 엔딩 기록과 모드 해금. 캠페인 rollback과 분리한다.

초기 autosave 주기는 약 30초의 simulation time으로 두되 설정값으로 관리한다. Protocol 완료, 중요한 Transaction 묶음 확정, app background 직전에도 autosave를 요청하며 연속 요청은 합친다. Failed 상태 자체를 정상 autosave로 덮어쓰지 않는다.

## 23.2 사용자 UX

SYSTEM에는 `SAVE NOW`를 둔다. 일반 플레이에서 슬롯 번호를 고르게 하지 않는다.

SYSTEM → RECOVERY에서는 최근 autosave 세대와 이전 Protocol Checkpoint를 볼 수 있다. 과거 checkpoint 복원 시 이후 진행이 사라진다는 확인을 받고, 선택 지점 뒤의 campaign/autosave/checkpoint를 폐기해 시간선이 섞이지 않게 한다.

## 23.3 저장해야 하는 것

simulationTick, Cells와 분열/wander 상태, Threats, Environment/Fields, Protocol state와 Preparation Timer, 실행된 Protocol Action, Cooldown, Resources, Research, Discovery, Trait Loadout, Mail Inbox/Queue, Narrative Flags, Emergence Evidence, Observation Records, RNG state를 저장한다.

## 23.4 저장하지 않는 것

Spatial Hash, 파티클, UI hover, 렌더 보간값처럼 현재 상태로 재구성 가능한 것은 저장하지 않는다.

## 23.5 Event Queue

단순 알림 큐는 저장하지 않는다. 미래 게임 결과에 영향을 주는 예약 작업은 상태 또는 scheduled command로 저장한다.

# 24. RNG

처음부터 과도하게 여러 RNG를 만들지 않는다.

최소:

```text
Gameplay RNG
Presentation RNG
```

를 분리한다.

UI 깜빡임, 노이즈, 파티클 같은 연출 랜덤이 게임 결과를 바꾸지 않게 한다.

필요해지면 Gameplay RNG 내부를 다음 substream으로 확장할 수 있다.

- Simulation
- Protocol
- Mutation

분리 수 자체가 목적은 아니다.

---

# 25. 성능 기준

Spatial Hash는 기본 도구이지 밀집 상태 성능을 자동으로 보장하는 해결책이 아니다.

## 25.1 MVP 성능 검증

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

## 25.2 장기 목표

`1,500 dynamic entities`는 하드캡이 아니라 성능 시험 기준이다.

실제 생성 한도는 별도 밸런스로 관리한다.

## 25.3 업데이트 주파수 분리

초기 예:

```text
Movement              20 Hz
근거리 감지            20 Hz
장거리 감지             5 Hz
Colony 재구성          2~5 Hz
Narrative Observation   2 Hz
```

실제 값은 프로파일링 후 정한다.

## 25.4 밀집 군집

모든 쌍의 거리를 계산하지 않는다.

가능하면:

- Spatial bucket
- 지역 중심점
- 군집 평균
- Field

등을 사용한다.

Object Pool, Worker, 병렬 처리 등은 프로파일링 결과가 필요할 때 도입한다.

---

# 26. 시각 규칙

레퍼런스 이미지는 하나의 완성 화면을 뜻하지 않는다.

용도를 분리한다.

| 레퍼런스 성격 | 적용 |
|---|---|
| Game vibe / UI | 기본 팔레트, 패널, 픽셀 밀도 |
| CRT 모니터 | 타이틀, 부팅, 종료 |
| 네온 로딩 | 진행 표시 |
| 글리치 계열 | 후반 이상 현상 |
| 현대 브랜드/서비스 문구 | 사용하지 않음 |

## 26.1 메인 플레이 화면

게임은 **세로 방향 고정**이다. FHD `1920×1080`을 세로로 돌린 **1080×1920(9:16)**을 디자인 기준으로 삼고, 픽셀 논리 화면은 **216×384**로 고정한다.

- `ResponsiveShell`이 기기의 safe area 안에 9:16 CRT 화면을 비율 유지로 최대 배치한다.
- 18:9, 19.5:9, 20:9처럼 더 긴 폰의 남는 영역은 CRT 케이스·터미널 여백·암부로 사용한다.
- 3:4 같은 넓은 세로 화면은 좌우 여백을 모니터 케이스로 처리한다.
- 게임 규칙과 배양 공간 논리 좌표는 기기 화면비 때문에 늘어나거나 잘리지 않는다.
- 노치, Dynamic Island, 상태 표시줄, 홈 인디케이터는 safe-area inset으로 피한다.
- UI는 상단 HUD / 중앙 배양 화면 / 하단 조작부 anchor zone으로 구성한다.
- 데스크톱 개발 창도 세로 9:16 CRT 화면을 가운데 표시한다.
- 터치는 physical → ResponsiveShell 역변환 → CRT 굴절 역변환 → 216×384 논리 좌표 순으로 변환한다.

CRT는 `Game/Ref/Ref 01.jpg`를 기준으로 하며 4색 녹색 팔레트, 픽셀 폰트, 주사선, 약한 번짐과 가장자리 암부를 사용한다. 강한 글리치는 후반 서사에 제한한다.

최소 화면비 테스트는 9:16, 9:18, 9:19.5, 9:20, 3:4 세로에서 수행한다.

## 26.4 타이틀과 부팅

`Game/Ref/타이틀 화면.txt`를 따른다.

1. `Ref 01` 스타일의 큰 도트 제목이 `D` → `o` → `t` → `.exe` 순서로 나타난다.
2. 제목 아래에 `TOUCH TO START`가 깜빡인다. 연출 중에 누르면 연출을 끝까지 넘긴다.
3. 누르면 `Ref 02` 스타일의 레트로 로딩 화면이 나온다. 문구는 `Dot.exe booting...`이다.
4. 로딩 연출이 끝나면 게임이 시작된다.

## 26.2 상태 구분

색과 밝기만으로 상태를 구분하지 않는다.

최소한 정지 화면에서도 다음이 구분돼야 한다.

- 정상 세포
- 적
- 감염
- 휴면

점멸은 보조 정보다.

분열과 위험 경고가 같은 “밝아짐” 하나로 표현되지 않도록 한다.

## 26.3 한글 UI

실제 목표 해상도에서 다음을 직접 검증한다.

- 최소 글자 크기
- 한글 픽셀 폰트 가독성
- Mail 본문
- 300개 이상 Cell 화면
- CRT 효과를 낮춘 상태에서 경고 구분

---

# 27. HUD 결정

`STABILITY %`는 메인 HUD에서 제거한다.

왜 84%인지 설명하기 어려운 복합 점수는 기본 정보로 쓰지 않는다.

대신 실제 상태를 보여준다.

예:

```text
CELLS
SYSTEM ENERGY
ENERGY OUTPUT
NUTRIENT RESERVE
CULTURE NUTRIENT
OXYGEN
CONTAMINATION
```

필요한 정보만 해당 시점에 노출한다.

OXYGEN과 CONTAMINATION은 해당 시스템이 들어오는 MVP 이후에 표시한다. Population Retention 목표가 있는 실험 중에는 `REQUIRED`(필요 개체 수)를 보여준다.

`STABILITY`라는 개념은 Protocol 평가나 연구 보고서에서 제한적으로 사용할 수 있다.

---

# 28. 스토리 진행률

`65~70%`, `75%` 같은 수치는 코드 조건으로 사용하지 않는다.

작가용 페이싱 목표일 뿐이다.

실제 Narrative 진행은 다음으로 결정한다.

```text
Narrative Phase
+
Required Observations
+
Required Previous Mail
+
Required Campaign Progress
```

초기 이상 징후와 후반의 명백한 자율 행동을 구분한다.

---

# 29. 엔딩 이후

엔딩 선택 결과는 영구 저장한다.

기본 흐름:

```text
Ending
→ Ending Record 저장
→ Title 복귀
→ EXPERIMENT MODE 해금
```

화면이 꺼지는 연출이 실제 애플리케이션 종료를 뜻하지 않는다.

---

# 30. 구현 환경

실제 게임 프로젝트 루트는 `Game/`이다. `.agents/`는 Luna Chat Coder 전용이며 게임 소스·문서·에셋을 두지 않는다.

출시 대상은 Google Play와 App Store다. TypeScript + Canvas/WebGL + Capacitor, Vite, Vitest를 사용한다.

- 세로 고정, 기준 디자인 1080×1920, 논리 화면 216×384.
- ResponsiveShell이 safe area와 화면비를 처리하고 simulation 좌표는 기기별로 바꾸지 않는다.
- 기본 입력은 터치이며 마우스도 같은 경로로 처리한다.
- 앱 background는 SYSTEM_SUSPEND로 pause하고 밀린 시간을 처리하지 않는다.
- 저장은 앱 전용 저장소에 하며 실패 시 직전 정상본을 보존한다.
- 게임 규칙과 `Game/src/features/`는 플랫폼을 모른다.

## 30.1 현지화

화면 글은 `Game/Locale/<언어>.json`의 키로 관리한다. 기준은 en.json, 1차 지원은 한국어·영어다.

## 30.2 단계별 테스트 절차

`Game/Docs/current/DEVELOPMENT_PIPELINE.html`을 따른다. 에이전트 배속 → 브라우저 배속 → 사람 테스트 순서이며 체크리스트는 `Game/Docs/tests/`에 둔다.

Stage 1 사람 테스트는 Chemotaxis OFF의 실제 P-01 초반과 P-01 완료 뒤 ON의 전후 비교를 모두 포함한다.

# 31. 바이브코딩 구조 규칙

콘텐츠 하나 추가 시 기존 시스템 `0~2개 수정`은 강제 규칙이 아니다.

다음 상황에서 건강한 구조를 판단하는 지표로만 사용한다.

- 새 연구 데이터
- 새 Mail
- 기존 행동을 사용하는 새 Threat 변형
- 새 Protocol 데이터

반대로 **새로운 근본 메커니즘**이 추가되면 다음이 함께 바뀌는 것은 정상일 수 있다.

- Domain
- Save
- UI
- Test
- System

정상적인 구조 변경까지 실패로 취급하지 않는다.

---

# 32. AGENTS.md / ARCHITECTURE

기존 `AGENTS.md`가 있다면 덮어쓰지 않는다.

기존 한국어 작성 규칙을 유지하면서 개발 규칙을 추가한다.

구현 착수 전에 다음 문서를 준비한다.

```text
AGENTS.md
ARCHITECTURE.md
CORE_GAME_RULES.md
```

권장 역할:

### AGENTS.md
AI 작업 규칙

### ARCHITECTURE.md
시스템 책임, 의존 방향, 이벤트, 저장 구조

### CORE_GAME_RULES.md
본 문서에서 확정한 게임 규칙의 정식 원본

---

# 33. MVP에서 의도적으로 미루는 기능

아래 기능은 현재 구현 규칙을 지나치게 복잡하게 만들기 때문에 MVP 이후로 미룬다.

- Mutation
- 개별 Cell Trait
- 지역 Oxygen Field
- FLOW
- ISOLATE
- 능동 Analysis
- 완전한 Emergence 캠페인
- UNKNOWN Mail 전체 시퀀스
- Virus
- Parasite
- Fungus
- 최종 Protocol
- 다중 역할 분화
- 전역 환경값(Temperature, pH, Oxygen)과 ENVIRONMENT 조작
- 폐기물, CONTAMINATION
- 군집 ID와 Cluster 생성
- 플레이어 SIGNAL
- 사전 반응(PRE_STIMULUS_RESPONSE) 메일. 예측 행동이 화면에서 실제로 일어나야 보낼 수 있는데 MVP에는 그 행동이 없다. MVP의 Phase 2 메일은 감염 실험 안내와 방어 연구에 대한 반응으로 한정한다.

MVP의 목적은 스토리 전체가 아니라 다음을 검증하는 것이다.

1. 한 점이 살아 움직이는 느낌이 있는가.
2. 세포 수가 늘어날수록 화면이 재미있어지는가.
3. 먹이 위치 조작이 플레이로 느껴지는가.
4. 연구가 실제 행동 변화를 만든다고 느껴지는가.
5. 다음 Protocol 준비가 전략적 선택을 만드는가.
6. Rapid Bacteria와의 첫 방어가 재미있는가.

---

# 34. 아직 밸런스 확정이 아닌 값

다음 수치는 규칙이 아니라 초기안이다.

- Safe Expression Limit 100
- Hard Expression Limit 125
- Simulation 20Hz
- Observation 2Hz
- Final Protocol 90/60/90초
- Population Retention 비율
- SYSTEM ENERGY harvest ratio
- Division cooldown
- Division duration
- Cell cap
- Threat cap
- Oxygen 소비량
- Nutrient 회복량

모두 Data/Balance 파일에서 바꿀 수 있게 한다.

---

# 35. 구현 전 필수 테스트 시나리오

## 에너지

- 최초 세포가 지속 분열해도 첫 유료 연구에 도달할 수 있다.
- Energy Storage 구매가 SYSTEM ENERGY 생산을 부자연스럽게 막지 않는다.
- 굶주리는 세포가 연구 자원을 계속 생산하지 않는다.
- 저장량이 가득 찬 세포의 넘친 몫이 SYSTEM ENERGY로 들어가지 않는다.
- 내부 에너지가 0인 세포는 Health가 줄고, 에너지를 되찾으면 회복한다.

## Protocol

- Preparing에 들어갈 때 Checkpoint가 정확히 한 번 생긴다.
- 수동 시작과 자동 시작이 같은 전환 함수를 거친다.
- 실패하면 Checkpoint를 불러와 Preparing으로 돌아오고 준비 타이머가 처음부터 돈다.
- Checkpoint 이후에 산 연구와 쓴 자원이 함께 되돌아간다. (연구만 남고 비용이 돌아오지 않는다.)
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
- FEED/SIGNAL/PURGE는 Pause 중 사용할 수 없다.
- Manual Pause 상태에서 Mail을 닫아도 자동 재개되지 않는다.

## Division

- 첫 분열 이후 부모도 divisionCooldown을 따른다.
- 갓 태어난 세포는 maturityAge와 divisionCooldown을 모두 채워야 분열한다.
- 개체 수 상한 직전에 여러 세포가 동시에 분열을 시작해도 상한을 넘지 않는다.
- PendingDeath 세포는 분열하지 않는다.
- 분열 도중 사망 시 자식이 생성되지 않는다.
- 분열 에너지 비용이 복제되지 않는다.

## Trait

- Safe Limit 초과는 허용된다.
- Hard Limit 초과는 거부된다.
- Trait 반복 변경 후 Modifier가 중복 누적되지 않는다.

## Save

- 저장 직후 Load하면 같은 상태를 복원한다.
- 같은 Save에서 같은 입력을 주고 일정 tick 진행 시 게임 결과가 재현된다.
- 이미 실행한 Protocol Action이 Load 후 다시 실행되지 않는다.
- one-shot Mail이 중복 수신되지 않는다.

## Campaign

- 대사/방어 중심 빌드에서도 대체 Evidence 경로를 통해 진행 가능하다.
- Current Capability가 없는데 History만으로 이상 행동 Mail이 발생하지 않는다.

## Performance

- Dense Colony 300 Cell + 증식 중인 적 상태를 검증한다.
- CRT 효과를 줄여도 Cell 상태 구분이 유지된다.

---

# 36. 최종 결정 요약

두 차례 검토를 거쳐 다음 원칙을 확정한다.

1. 발현 부하는 **안전 한도와 절대 상한**을 분리한다.
2. 연구용 ENERGY는 저장 잉여가 아니라 **대사 순생산의 일부**로 얻는다.
3. Protocol 목표는 단일 생존율이 아니라 **명시적 Objective 조합**으로 판정한다.
4. Checkpoint는 버튼이 아니라 **Preparing 진입**에서 생성한다.
5. 실패하면 게임 오버가 되고 **Checkpoint를 통째로 불러와 준비 단계부터** 다시 한다.
6. 발견은 한 번만 기록하고 DATA는 바로 지급한다. 미확정 DATA는 두지 않는다.
7. Simulation Time과 UI Transaction을 분리한다.
8. Pause 중 연구/메일/저장은 가능하지만 FEED/SIGNAL/PURGE는 금지한다.
9. Tick 처리에서 계산과 확정을 분리하고 사망 결과 이후 Protocol을 판정한다.
10. Trait은 MVP에서 **전역 Loadout**으로 관리한다.
11. 세포 LifeMode, Health, Infection을 서로 다른 축으로 관리한다.
12. 생애 나이와 재분열 Cooldown을 분리한다.
13. MVP는 실제 효과가 연결된 연구 10개와 Protocol 3개로 검증한다.
14. Emergence는 무한 누적 점수보다 **고유 행동 Evidence**를 사용한다.
15. 이상 Mail은 실제 행동 → Observation → Narrative → Mail 순서로만 발생한다.
16. 최종 Protocol은 단순 버티기가 아니라 **무개입 균형·충격·회복**을 검증한다.
17. 저장은 재현 가능한 시뮬레이션을 이어갈 수 있는 상태를 보존한다.
18. Spatial Hash만으로 성능 문제를 해결했다고 보지 않는다.
19. CRT/글리치는 미감보다 **판독성**을 우선하며 강한 효과는 제한적으로 쓴다.
20. 아직 검증되지 않은 숫자는 규칙이 아니라 **밸런스 데이터**로 취급한다.

---

기획서 v0.3과 개발 명세서 v0.2는 이 문서에 맞춰져 있다.

---

# 변경 기록

## v1.1 (2026-09-19) — 3차 검토 결정

근거: `Dot.exe_기획서v0.2_3차검토.md`

| 절 | 바뀐 내용 |
|---|---|
| 3.3 | 과발현 패널티의 “분열 비용 또는 분열 준비 시간”을 분열 비용으로 확정 |
| 4.5~4.7 | 분열 에너지 반분, 저장량이 찼을 때와 순생산이 음수일 때의 처리, 굶주림의 Health 감소 추가 |
| 5.2 | Population Retention 기준을 `referencePopulation`만으로 변경. HUD에 필요 개체 수 표시 |
| 6~7 | 체크포인트를 Preparing 진입 시점으로 이동. 실패는 게임 오버 후 체크포인트 전체 로드. RetryPreparation 삭제. 영구 진행/실험 상태 구분 삭제. 연구 구매·Trait 변경은 Running에서만 금지. Running이 아닐 때의 전멸은 초기 세포 재생성. Completed 시 남은 위협 제거 |
| 8, 16 | 발견·보상 분리와 Pending DATA 삭제. DATA는 바로 지급 |
| 9 | ANALYSIS와 GAME OVER 화면도 Pause. Pause 중 명령 금지는 악용 차단이 아니라 마찰임을 명시 |
| 10 | 분열 진행도를 16단계에서 갱신. 경쟁 자원은 요청량 비례 분배. 위협 개체의 처리 단계 명시 |
| 12 | 분열 시간 조건을 하나로 통일. 기본 조건에서 지역 밀도 제외. 개체 수 상한 검사 시점 추가 |
| 13.1 | Trait 변경은 Running에서만 금지 |
| 14~15 | Energy Storage의 MVP 대가, Chemotaxis 무상 지급, Adhesion은 결속력만, P-03 “제한”의 뜻, MVP 조작은 FEED와 PURGE |
| 17~18 | 전역 환경값·폐기물·Cluster 생성을 MVP에서 제외. 부착은 군집 없는 세포끼리도 작용 |
| 19.1 | Evidence는 체크포인트 로드 때만 되돌아감 |
| 23 | 체크포인트는 별도 저장본. 엔딩 기록은 프로필 데이터 |
| 27, 33, 35, 36 | 위 변경에 맞춰 HUD, 미루는 기능, 테스트 시나리오, 요약 갱신 |
| 30.1 | 현지화 규칙 추가. 화면의 글은 `Game/Locale/*.json`에서 키로 가져온다 |

## v1.2 (2026-09-19) — 출시 플랫폼과 테스트 절차

| 절 | 바뀐 내용 |
|---|---|
| 30 | 1차 플랫폼을 Windows 데스크톱에서 Google Play · App Store(유료 ₩1,500, Capacitor)로 변경. 가로 고정, 터치 입력, 기준 기기, 저장 위치 |
| 30.2 | 단계별 3차 테스트 절차(에이전트 배속 → 브라우저 배속 → 사람 체크리스트) 추가. 헤드리스 실행과 배속 설정 요구 |

## v1.3 (2026-09-19) — 세로 화면과 도트·CRT 화면 규칙

| 절 | 바뀐 내용 |
|---|---|
| 26.1 | “강한 CRT 왜곡을 상시 쓰지 않는다”를 뒤집었다. 굴절과 모니터 틀을 항상 적용한다. 저해상도 논리 화면, 4색 팔레트, 픽셀 폰트, 가장자리 들여 놓기, 터치 좌표의 굴절 보정을 규칙으로 추가 |
| 26.4 | 타이틀과 부팅 흐름 추가 (`Game/Ref/타이틀 화면.txt`) |
| 30 | 가로 고정을 세로 고정으로 변경. 논리 화면 가로 216픽셀 |


## v1.4 (2026-09-19) — 프로젝트 구조·명칭·모바일 UX 정리

- 작품명과 극중 시스템명을 Dot.exe로 통합.
- .agents와 Game을 분리하고 current/history 문서 체계를 확정.
- 1080×1920 기준, 216×384 논리 화면, ResponsiveShell 적응 규칙 확정.
- Briefing → 제한시간 Preparing → Running 흐름으로 준비 긴장 설계.
- P-01 프로토타입을 Chemotaxis OFF → 완료 보상 후 ON으로 본편과 일치.
- Campaign 1슬롯 + Rolling Autosave 3세대 + Protocol Checkpoint + ProfileData 저장 UX 확정.
