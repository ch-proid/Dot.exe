# Dot.exe CORE GAME RULES

> 문서 버전: 1.3 (2026-09-19)  
> 이 문서가 게임 규칙의 **정식 원본**이다. 기획서와 명세서의 `결정 N절`, `정본 N절`은 이 문서의 N절을 가리킨다.  
> 규칙을 바꿀 때는 이 문서를 먼저 고치고 문서 끝 `변경 기록`에 적은 뒤 기획서·명세서·코드를 맞춘다.  
> 아직 플레이테스트를 거치지 않은 수치는 `초기 밸런스 값`이며 고정 규칙이 아니다. 실제 값은 `src/data/balance/`에 둔다.  
> 출처: 세 차례 검토의 결정(`Docs/Dot.exe_두차례_검토_결정사항.md`, v1.2에서 동결).

---

# 1. 문서 우선순위

게임 규칙이 여러 문서에서 다르게 적혀 있을 경우 다음 순서로 해석한다.

1. 이 문서(`CORE_GAME_RULES.md`)
2. 개발 명세서
3. 상세 기획서
4. 밸런스 데이터
5. 예시 문구 및 연출용 수치

동일한 규칙을 여러 문서에 복사해 관리하지 않는다.  
정식 규칙은 한 곳에만 두고 다른 문서에서는 해당 규칙을 참조한다.

---

# 2. 작품명과 극중 프로그램명

- **작품명:** `Dot.exe`
- **극중 프로그램명:** `CULTURE//SYS`

두 이름을 혼용하지 않는다.

`Dot.exe`는 게임 전체의 제목이고, `CULTURE//SYS`는 플레이어가 연구실 컴퓨터에서 실행하는 극중 배양 시스템 소프트웨어다.

---

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

실패 처리는 일반적인 싱글플레이 게임의 방식을 따른다. 실패하면 게임 오버가 되고 체크포인트를 통째로 다시 불러온다.

## 6.1 상태 흐름

```text
Idle
→ Preparing
→ Running
→ Completed
→ 다음 프로토콜의 Preparing (없으면 Idle)

Running
→ Failed
→ GAME OVER 화면
→ 체크포인트 로드
→ Preparing
```

`RetryPreparation` 상태는 두지 않는다.

Idle에서도 시뮬레이션 시간은 흐른다.

## 6.2 체크포인트 생성 시점

프로토콜이 `Preparing`에 들어가는 순간 정확히 한 번 생성한다. 준비 타이머가 돌기 시작하는 시점이다.

체크포인트는 그 시점의 전체 저장 데이터다. 일반 저장과 같은 형식을 쓴다.

이전 프로토콜의 체크포인트도 지우지 않고 남겨 둔다. 준비 단계를 다시 해도 넘을 수 없을 만큼 배양체가 무너졌다면 더 앞의 체크포인트를 불러올 수 있다.

실험 시작 직전이 아니라 준비 단계 시작에 두는 까닭은, 약한 상태로 실험에 들어갔다가 실패했을 때 같은 상태로 되돌아가 막히는 일을 없애기 위해서다. 준비 단계부터 다시 하면 연구, 특성 구성, 증식을 모두 다시 할 수 있다.

## 6.3 Running 전환

수동 시작과 자동 시작 모두 같은 전환 함수를 사용한다.

```text
수동 시작
Start Button
→ transitionToRunning()

자동 시작
Preparation Timer == 0
→ transitionToRunning()
```

## 6.4 실패 후 복구

전멸이든 필수 목표 미달이든 실패는 같은 방식으로 처리한다.

```text
CULTURE FAILURE

RESTORING
LAST VIABLE SAMPLE...
```

체크포인트를 불러오면 준비 타이머가 처음부터 다시 돈다.

---

# 7. 복원 범위와 연구 구매

## 7.1 복원 범위

체크포인트를 불러오면 **전부** 그 시점으로 돌아간다.

- 세포, 적, 환경 Field
- SYSTEM ENERGY, DATA, NUTRIENT RESERVE
- 연구 해금, Trait Loadout
- Protocol 상태, Cooldown, 일시 효과
- 메일, Narrative Flag, Observation Record, 발견 기록, Emergence Evidence
- simulationTick, RNG 상태

“영구 진행”과 “실험 상태”를 나누지 않는다. 실패한 시도에서 받은 메일과 발견은 다시 하면서 다시 얻는다.

체크포인트와 무관하게 유지하는 것은 엔딩 기록과 모드 해금 정보뿐이다. 이 둘은 캠페인 저장과 별도로 보관한다.

## 7.2 연구 구매와 Trait 변경

`Running` 상태에서만 금지한다. Idle, Preparing, Completed에서는 허용한다.

## 7.3 Running이 아닐 때의 전멸

Running이 아닐 때 Viable Cell이 0이 되면 게임 오버로 처리하지 않는다.

`RESTORING LAST VIABLE SAMPLE` 연출과 함께 초기 세포 하나를 배양 공간 중앙에 다시 만든다. 자원과 연구는 그대로 둔다.

P-01 시작 전에 첫 세포를 굶겨 죽이는 실수를 받아 주기 위한 규칙이다.

## 7.4 프로토콜 완료 시

`Completed`로 넘어갈 때 남아 있는 위협 개체를 제거한다. 실험이 끝난 뒤 남은 세균에게 배양체가 전멸하는 일을 막는다.

---

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

`Simulation Time`과 `UI/Transaction Processing`을 분리한다.

## 9.1 Simulation Time이 멈추는 화면

CULTURE 외의 전체 화면은 모두 시뮬레이션을 Pause한다.

- MAIL
- RESEARCH
- ANALYSIS
- SYSTEM
- GAME OVER
- 명시적인 Pause

Pause 시 멈추는 것:

- 세포 시뮬레이션
- 적 행동
- Protocol Timer
- SYSTEM ENERGY 생산
- NUTRIENT 보충
- Cooldown
- 환경 시간 변화

## 9.2 Pause 중 즉시 처리 가능한 명령

다음은 시뮬레이션 시간이 멈춰도 즉시 검증·확정할 수 있다.

- 연구 구매 (Running이 아닐 때)
- Trait Loadout 변경 (Running이 아닐 때)
- 메일 읽음 처리
- 설정 변경
- 저장

## 9.3 Pause 중 금지되는 시뮬레이션 명령

다음은 Pause 상태에서 사용할 수 없다.

- FEED
- SIGNAL
- PURGE
- 환경 조작
- 전투성 개입

Pause 상태에서 명령을 예약해 두는 플레이에 마찰을 주기 위한 규칙이다. Pause → 위치 확인 → 재개 → 곧바로 클릭까지 막지는 않으며, 그것을 막는 장치를 더 얹지 않는다.

## 9.4 Pause Source

단순 boolean 하나를 사용하지 않는다.

예:

```text
MANUAL_PAUSE
MAIL_MODAL
RESEARCH_MODAL
ANALYSIS_MODAL
SYSTEM_MODAL
GAME_OVER
```

여러 이유가 동시에 존재할 수 있다.

수동 Pause 상태에서 Mail을 열었다 닫았다고 자동으로 재개되지 않는다.

## 9.5 오프라인 진행

기본 캠페인에서는 오프라인 진행을 사용하지 않는다.

---

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

목적:

- 기본 먹이 공급
- 이동
- 대사
- 분열
- 첫 성장 경험

초기에는 Chemotaxis가 없다.

플레이어는 세포 가까이에 영양분을 공급해야 한다.

P-01을 완료하면 Chemotaxis가 연구 완료 상태로 지급되고 바로 활성화된다. “먹이를 찾아가는 행동”이 눈에 띄는 성장 보상이 된다.

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

저장은 **시뮬레이션 업데이트나 Transaction 처리 중이 아닌 일관된 상태**에서 수행한다.

반드시 새 tick을 한 번 돌린 뒤 저장할 필요는 없다.

## 23.1 저장해야 하는 것

최소:

- simulationTick
- Cells
- Cell division progress
- lastDivisionTick
- wander state
- Threats
- Environment
- Scalar Fields
- Protocol state
- 이미 실행한 Protocol Action
- Cooldowns
- Isolation state
- Resources
- Research
- Discovery Records
- Trait Loadout
- Mail Inbox
- Mail Queue
- Narrative Flags
- Emergence Evidence
- Observation Records
- RNG state

tick 기준 값(`lastDivisionTick`, Cooldown 등)은 `simulationTick`과 함께 저장·복원되므로 절대 tick으로 두어도 어긋나지 않는다.

체크포인트는 저장 데이터 안에 넣지 않는다. 같은 형식의 별도 저장본으로 둔다. (6.2)

엔딩 기록과 모드 해금 정보는 캠페인 저장과 분리된 프로필 데이터에 둔다.

## 23.2 저장하지 않는 것

현재 상태로 재구성 가능한 값:

- Spatial Hash
- 화면 파티클
- UI hover
- 렌더 보간값

## 23.3 Event Queue

단순 알림성 이벤트 큐는 저장하지 않는다.

미래 게임 결과에 영향을 주는 예약 작업은:

- 이미 상태로 확정하거나
- 별도 scheduled command로 저장한다.

---

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

게임 전체가 낡은 CRT 모니터 안에서 돌아가는 것처럼 보인다. 기준은 `Ref/Ref 01.jpg`다.

- 화면의 가운데는 평면이고 끝쪽에서만 약하게 굴절된다. 둘레에 레트로 모니터의 틀이 있다. 타이틀부터 엔딩까지 항상 적용한다.
- 분위기는 어둡고 조금 무섭다. 색감의 기준은 `Ref/색감 예시.png`다: 검정에 가까운 바탕, 가라앉은 선, 네온 에메랄드 글자, 어둠에 잠기는 가장자리. 똑같이 만들 필요는 없다.
- UI는 면을 채운 밝은 패널 대신 가는 테두리와 글자로 만든다. 밝은 색은 강조할 것과 세포에만 쓴다.
- 주사선, 픽셀 격자, 약한 번짐, 가장자리 어두움을 함께 쓴다.
- 모든 화면은 저해상도 논리 화면(가로 216픽셀)에 그린 뒤 정수 도트로 키운다. 부드러운 확대, 안티앨리어싱, 그러데이션을 쓰지 않는다.
- 색은 4색 녹색 팔레트뿐이다. 중간색은 가장 가까운 팔레트 색으로 맞춘다. 음영은 색을 늘리지 않고 체크무늬 같은 도트 패턴으로 낸다.
- 글자는 픽셀 폰트(Galmuri11, OFL 1.1)를 제 크기 그대로 쓴다. 큰 글자는 도트를 정수배로 키워 만든다.
- 도트 밀도는 `Ref/Game vibe ref.jpg`를 따른다. `Ref/UI Ref.jpg`는 아이콘과 부품의 도트 표현만 참고한다.

굴절 때문에 화면 가장자리와 모서리가 가려진다. UI는 가장자리에서 안쪽으로 들여 놓고, 터치 좌표는 굴절과 같은 식을 거쳐 논리 좌표로 바꾼다. 눈에 보이는 위치와 눌리는 위치가 어긋나면 안 된다.

효과의 세기는 설정에서 줄일 수 있어야 하고, 줄여도 26.2의 상태 구분이 유지돼야 한다. 굴절과 틀은 남긴다.

글리치와 강한 왜곡은 후반 서사 이벤트에 제한한다.

## 26.4 타이틀과 부팅

`Ref/타이틀 화면.txt`를 따른다.

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

출시 대상은 **Google Play와 App Store**다. 유료 다운로드(₩1,500)이며 인앱결제, 광고, 데이터 수집이 없다.

게임 로직은 TypeScript로 작성하고 Canvas/WebGL 계열로 그린다. 개발 중에는 데스크톱 브라우저에서 실행하고, 출시할 때 Capacitor로 Android와 iOS 앱으로 감싼다. 빌드 도구는 Vite, 테스트는 Vitest다.

- 화면은 세로 고정이다. 논리 화면의 가로는 216픽셀로 고정하고 세로는 기기 비율에 맞춘다. 넓은 창에서는 9:16 비율로 가운데에 놓는다.
- 기본 입력은 터치다. 개발과 테스트를 위해 마우스 입력도 같은 경로로 받는다.
- hover에 기대는 UI를 만들지 않는다.
- F1~F5 표기는 연출로만 남긴다. 화면은 탭 버튼으로 연다. 키보드가 있으면 숫자키를 보조 단축키로 쓴다.
- 앱이 뒤로 가면 Pause한다. 돌아올 때 밀린 시간을 처리하지 않는다. (9.5)
- 성능 기준 기기는 3~4년 된 중급 Android 기기 한 대로 정하고, 25절의 측정은 그 기기에서 한다. 기기는 Stage 1 관문 전에 확정한다.
- 저장은 앱 전용 저장소에 하고, 쓰기에 실패하면 이전 저장본을 보존한다.

게임 규칙과 `features/`는 플랫폼을 모른다. 플랫폼에 따라 바뀌는 것은 `presentation/`, `input/`, 저장 매체뿐이다.

## 30.2 단계별 테스트 절차

개발은 `Docs/Dot.exe_개발_파이프라인.html`의 단계를 하나씩 밟는다. 한 단계가 끝나면 다음 세 차례 테스트를 순서대로 거치고, 사람이 승인해야 다음 단계로 간다.

1. **에이전트 배속 테스트**: 자동 테스트 전체와, 시뮬레이션을 화면 없이 빠르게 돌려 규칙을 확인하는 헤드리스 시나리오.
2. **브라우저 배속 테스트**: 메인 에이전트가 Chrome에 실제 화면을 띄워 배속으로 돌려 보고 화면, 콘솔 오류, 조작을 확인한다.
3. **사람 테스트**: 단계마다 `Docs/테스트/`에 체크리스트를 만들어 공유하고, 사람이 직접 해 보고 판정한다. 재미와 느낌에 대한 관문은 사람만 판정한다.

이를 위해 시뮬레이션은 화면 없이 돌릴 수 있어야 하고, 개발 빌드에는 배속 설정이 있어야 한다. 배속은 tick을 더 자주 돌리는 것이며 tick 하나의 계산은 바꾸지 않는다.

## 30.1 현지화

화면에 나오는 글은 코드와 콘텐츠 데이터에 직접 적지 않는다. 키만 적고 글은 `Locale/<언어 코드>.json`에서 가져온다.

- 기준 파일은 `en.json`이다. 1차 지원 언어는 한국어와 영어다.
- 새 언어는 JSON 파일 하나를 더하는 것으로 끝나야 한다.
- 연구·특성·프로토콜·메일 데이터는 `title`, `body` 같은 문자열 대신 키를 갖는다.
- 언어에 따라 달라지는 것은 글뿐이다. 게임 규칙과 서사 조건은 언어와 무관하다.
- 폰트는 언어별로 지정할 수 있게 한다. (26.3)

---

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
| 30.1 | 현지화 규칙 추가. 화면의 글은 `Locale/*.json`에서 키로 가져온다 |

## v1.2 (2026-09-19) — 출시 플랫폼과 테스트 절차

| 절 | 바뀐 내용 |
|---|---|
| 30 | 1차 플랫폼을 Windows 데스크톱에서 Google Play · App Store(유료 ₩1,500, Capacitor)로 변경. 가로 고정, 터치 입력, 기준 기기, 저장 위치 |
| 30.2 | 단계별 3차 테스트 절차(에이전트 배속 → 브라우저 배속 → 사람 체크리스트) 추가. 헤드리스 실행과 배속 설정 요구 |

## v1.3 (2026-09-19) — 세로 화면과 도트·CRT 화면 규칙

| 절 | 바뀐 내용 |
|---|---|
| 26.1 | “강한 CRT 왜곡을 상시 쓰지 않는다”를 뒤집었다. 굴절과 모니터 틀을 항상 적용한다. 저해상도 논리 화면, 4색 팔레트, 픽셀 폰트, 가장자리 들여 놓기, 터치 좌표의 굴절 보정을 규칙으로 추가 |
| 26.4 | 타이틀과 부팅 흐름 추가 (`Ref/타이틀 화면.txt`) |
| 30 | 가로 고정을 세로 고정으로 변경. 논리 화면 가로 216픽셀 |
