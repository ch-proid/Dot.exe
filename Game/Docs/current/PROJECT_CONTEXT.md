# Dot.exe 프로젝트 맥락

> 문서 역할: 이 파일은 Dot.exe 프로젝트의 **지속 맥락 문서**다.  
> 마지막 갱신: 2026-09-19  
> 현재 작업 브랜치: `docs/project-normalization`  
> 관련 PR: #1 `Normalize Dot.exe project structure and core mobile UX rules`
>
> 이 문서는 채팅 기록을 대신해 “왜 현재 설계가 이렇게 되었는지”를 빠르게 복구하기 위한 파일이다.  
> **게임 규칙 자체의 최종 정본은 항상 `CORE_GAME_RULES.md`다.** 이 문서와 정본이 충돌하면 정본을 따른다.
>
> 앞으로 Dot.exe 작업을 시작할 때는 이 파일을 먼저 읽고, 이어서 `CORE_GAME_RULES.md` → `ARCHITECTURE.md` → 해당 기획/명세 문서를 읽는다.

---

# 1. 프로젝트 한 줄 정의

**Dot.exe는 세로형 모바일 생명 시뮬레이션 게임이다.**

플레이어는 1990년대풍 연구실 컴퓨터에서 정체불명의 배양 실험을 진행한다. 처음에는 단순한 세포 배양 게임처럼 보이지만, 연구와 실험이 진행될수록 세포가 단순한 생명체가 아니라 **컴퓨터 안에서 살아가는 인공 생명**일 수 있다는 정황이 나타난다.

플레이어가 느껴야 하는 감정의 큰 흐름은 다음과 같다.

```text
성장시키는 즐거움
→ 관찰하는 재미
→ 다음 실험을 준비하는 긴장
→ 예상 밖 행동의 발견
→ 불안
→ “내가 이것들을 만들고 있는 것 아닌가?”라는 의심
→ 플레이어와 세포의 관계 역전
→ “우리 세계도 누군가의 규칙 안에 있는가?”라는 여운
```

중요한 설계 모토는 **모든 기능에 의미와 감정을 부여하는 것**이다.

- 버튼 하나를 누르는 이유가 있어야 한다.
- 연구 하나가 단순 수치 상승이 아니라 행동 변화를 만들어야 한다.
- 자원 하나도 플레이어의 판단을 바꿔야 한다.
- UI는 정보 표시를 넘어 “낡은 연구 장비를 운용한다”는 감각을 만들어야 한다.
- 후반 공포는 갑자기 점프스케어처럼 등장하지 않고, 플레이어가 오래 관찰한 생명체가 예상 밖 행동을 하면서 서서히 생겨야 한다.

---

# 2. 현재 프로젝트 상태

아직 본격 게임 구현에 들어가기 전 단계다.

현재 저장소에는 다음이 중심이다.

- 게임 규칙 정본
- 상세 기획
- 개발 명세
- 아키텍처
- 개발 파이프라인
- 한국어 번역 기준
- 과거 검토 기록
- 시각 레퍼런스
- HTML UI 프로토타입

즉 현재 단계의 목적은 **실개발 전에 규칙·UI·용어·캠페인 구조를 닫는 것**이다.

현재까지 합의한 큰 순서는:

```text
문서/규칙 정리
→ UI 방향 확정
→ MVP 이후 스토리·캠페인·메일 전체 확정
→ 구현 데이터 확정
→ Milestone 1 개발 시작
```

이다.

---

# 3. 저장소 구조와 Luna 분리

초기 저장소에는 Luna Chat Coder 파일과 Dot.exe 게임 자료가 루트에서 섞여 있었다.

이를 다음처럼 분리했다.

```text
/
├─ .agents/
│  ├─ LICENSE
│  └─ skills/luna-chat-coder/
│
├─ Game/
│  ├─ AGENTS.md
│  ├─ README.md
│  ├─ Docs/
│  │  ├─ current/
│  │  └─ history/
│  ├─ Design/
│  ├─ Locale/
│  └─ Ref/
│
├─ AGENTS.md
├─ README.md
└─ README.ko.md
```

원칙:

- `.agents/` = Luna Chat Coder 통합 전용
- `Game/` = Dot.exe 게임 전용
- 게임 코드·문서·레퍼런스는 `.agents/`에 넣지 않는다.
- Luna 구현 파일은 `Game/`에 넣지 않는다.
- Luna의 MIT 라이선스는 루트가 아니라 `.agents/LICENSE`로 이동했다.
- Dot.exe 게임 자체 라이선스는 별도 결정 전까지 명시하지 않는다.

---

# 4. 현행 문서와 과거 기록

현행 문서는 `Game/Docs/current/`에 둔다.

현재 핵심 문서:

- `CORE_GAME_RULES.md` — 게임 규칙의 최상위 정본
- `ARCHITECTURE.md` — 시스템 책임, 의존 구조, 저장, 시간 처리
- `GAME_DESIGN.md` — 플레이 경험, 콘텐츠, 서사 방향
- `DEVELOPMENT_SPEC.md` — 실제 구현 명세
- `DEVELOPMENT_PIPELINE.html` — 단계별 개발 및 검증 절차
- `LOCALIZATION_KO_REFERENCE.md` — 한국어 용어 검토표
- `PROJECT_CONTEXT.md` — 이 문서. 프로젝트 결정의 맥락과 다음 작업을 연결

과거 검토와 결정 기록은 `Game/Docs/history/`에 둔다.

과거 기록은 **왜 현재 규칙이 만들어졌는지 추적하기 위한 자료**이며 구현 기준이 아니다.

문서 충돌 시 우선순위:

```text
CORE_GAME_RULES
→ DEVELOPMENT_SPEC
→ GAME_DESIGN
→ balance/data
→ 연출 예시
```

---

# 5. 명칭 통합

초기 문서에서는 게임 제목이 `Dot.exe`, 극중 프로그램명이 `CULTURE//SYS`였다.

이 구분은 제거했다.

현재 확정:

- 작품명: **Dot.exe**
- 극중 플레이어가 실행하는 시스템: **Dot.exe**
- 타이틀/부팅/시스템 헤더: **Dot.exe**

`CULTURE`, `RESEARCH`, `MAIL`, `ANALYSIS` 등은 별도 프로그램명이 아니라 기능/화면 이름으로 사용 가능하다.

현행 문서와 과거 기록에 남아 있던 `CULTURE//SYS` 표기는 모두 정리했다.

---

# 6. 플랫폼과 화면 방향

현재 방향은 **모바일 세로 고정**이다.

기준:

- 디자인 기준 해상도: **1080×1920**
- 논리 픽셀 화면: **216×384**
- 입력: 터치
- 개발 중 마우스는 같은 입력 경로로 지원
- Android/iOS
- TypeScript + Canvas/WebGL + Capacitor
- Vite + Vitest

다양한 기기 화면비에서 게임 영역 자체를 늘이거나 잘라 시야를 바꾸지 않는다.

`ResponsiveShell`이 다음을 처리한다.

- 9:16 기준 게임 화면 유지
- 18:9 / 19.5:9 / 20:9 대응
- 태블릿형 3:4 세로 대응
- 노치 / Dynamic Island / 상태 표시줄 / 홈 인디케이터 safe area
- 남는 공간을 CRT 케이스, 터미널 여백, 암부로 자연스럽게 처리

기기 화면비가 게임 난이도나 배양 공간 크기를 바꾸면 안 된다.

---

# 7. 비주얼 방향

전체 게임은 오래된 CRT 연구 장비의 화면처럼 보여야 한다.

핵심:

- 거의 검은 짙은 녹색 바탕
- 제한된 녹색 팔레트
- 아주 얇은 녹색 계측선
- 픽셀 폰트
- 스캔라인
- 약한 CRT 곡률
- 화면 가장자리의 약한 왜곡
- 낡은 화면 같은 비네팅
- 후반으로 갈수록 점점 증가하는 이상 신호

강한 글리치를 평상시에 계속 사용하지 않는다.

평소 글리치는:

- 수 초~십여 초 간격
- 약 0.1초 수준
- 얇은 수평 노이즈
- 1px 정도의 순간 위치 어긋남

수준으로 사용한다.

목적은 “화면이 화려하게 고장 난다”가 아니라:

> 방금 화면이 이상했던 것 같은데?

라는 작은 의심이다.

---

# 8. 폰트

사용자가 직접 제시한 후보:

- 한국어: Galmuri
- 영어: Shylock NBP

검토 결과 현재 방향은 **Galmuri11 하나로 한국어/영문을 모두 처리**하는 것이다.

이유:

- 픽셀 느낌이 분명하다.
- 한국어와 영문을 한 글꼴로 통일할 수 있다.
- 화면 전체의 시대감을 안정적으로 유지할 수 있다.
- OFL 계열 사용 조건으로 프로젝트 관리가 단순하다.

실제 게임 구현에서는 폰트 파일을 프로젝트에 적절히 포함하고 라이선스 고지를 유지한다.

---

# 9. 핵심 게임 루프

현재 큰 루프:

```text
세포 유지·증식
→ ENERGY 생산
→ 연구/특성 해금
→ 다음 프로토콜 확인
→ 준비
→ 프로토콜 실행
→ 감염/기아/독성/자원 경쟁 등 대응
→ 생존
→ DATA / 발견 / 연구 해금
→ 더 복잡한 행동
→ 다음 실험
```

핵심 철학은 숫자보다 **행동의 변화**다.

예:

- Chemotaxis를 얻으면 단순히 이동속도 +10%가 아니라 먹이를 찾아가는 행동이 생긴다.
- Alarm Signal을 얻으면 적을 본 한 세포의 정보가 주변으로 전달된다.
- Adhesion은 “방어 +5”가 아니라 개체들이 서로 붙는 방식 자체를 바꾼다.

---

# 10. 자원 명칭

초기에는 `SYSTEM ENERGY`라는 이름을 사용했다.

사용자 결정으로 **ENERGY**로 통일했다.

현재:

- `ENERGY` / 한국어 `에너지`
- `DATA` / 한국어 `데이터`
- `NUTRIENT RESERVE` / 한국어 `영양 비축량`
- 개별 세포가 가진 값은 `CELL ENERGY` / `세포 내부 에너지`

내부 코드 키도 `systemEnergy`가 아니라 `energy`를 사용한다.

ENERGY는 세포 내부 에너지와 다른 전역 자원이다.

세포가 영양을 먹어 대사한 뒤 생존에 필요한 몫을 우선 사용하고, 남는 순생산 일부가 ENERGY가 된다.

---

# 11. 세포와 이동

세포 위치는 타일이나 격자가 아니다.

**연속적인 2D 좌표와 속도**를 사용한다.

따라서 비주얼이 한 픽셀짜리 사각형이어도:

- 이동은 자유롭고 부드럽다.
- 한 칸씩 끊어서 이동하지 않는다.
- 렌더링 모양만 도트다.
- Spatial Hash는 충돌/근접 검색 최적화일 뿐 게임 이동 격자가 아니다.

현재 UI 방향에서 특히 중요한 표현 규칙:

- 세포 = **하얀색 정사각 픽셀 1칸**
- 영양분 = **세포보다 작은 초록색 정사각 픽셀**
- 둥근 원형 점 표현은 사용하지 않는다.

이 규칙은 사용자가 최신 UI 검토에서 명확히 요청한 방향이다.

---

# 12. 연구와 Trait

연구는 능력을 해금하고, Trait Loadout은 실제로 어떤 능력을 발현할지 결정한다.

MVP에서는 개별 세포별 Trait가 아니라 **배양체 전체에 적용되는 전역 Loadout**이다.

발현 부하(Expression Load)가 있으며 너무 많은 Trait을 동시에 켜면 대가가 생긴다.

현재 전체 연구 계통:

- 대사
- 증식
- 이동·감지
- 방어
- 군집
- 신호 전달

현재 문서에 정의된 연구/특성은 54개이며 한국어 이름은 `LOCALIZATION_KO_REFERENCE.md`에 정리되어 있다.

MVP 연구 10개:

1. Enhanced Glycolysis / 해당작용 강화
2. Energy Storage / 에너지 저장
3. Accelerated Mitosis / 분열 가속
4. Density Sensing / 밀도 감지
5. Chemotaxis / 화학주성
6. Hazard Avoidance / 위험 회피
7. Reinforced Membrane / 강화 세포막
8. Phagocytosis / 식균 작용
9. Alarm Signal / 경보 신호
10. Adhesion / 세포 접착

한국어 명칭은 아직 사용자 최종 검토 전이므로 변경 가능하다.

특히 검토가 필요한 후보:

- 해당작용 강화 ↔ 에너지 대사 강화
- 계획 세포사 ↔ 자가 사멸
- 반응 조건화 ↔ 학습 반응

---

# 13. P-01과 프로토타입 일치

초기 명세에는 문제가 있었다.

- 실제 P-01: Chemotaxis 없이 시작
- 프로토타입: Chemotaxis를 켜고 검증

이러면 프로토타입이 실제 첫 플레이 경험을 검증하지 못한다.

현재 수정된 방향:

```text
Chemotaxis OFF
→ 세포 가까이에 FEED
→ 기본 이동
→ 섭취
→ 분열
→ P-01 완료
→ Chemotaxis 연구 완료 상태로 지급
→ 즉시 활성
→ 더 먼 위치에 FEED
→ 세포들이 스스로 먹이를 찾아감
```

P-01의 첫 큰 보상은 “수치 증가”가 아니라 **같은 FEED 조작의 의미가 달라지는 것**이다.

Milestone 1에서도 다음 두 질문을 별도로 통과해야 한다.

1. Chemotaxis 없이도 초반 관찰/먹이 공급/분열이 재미있는가?
2. Chemotaxis를 얻은 순간 “진화했다”는 체감이 분명한가?

---

# 14. 프로토콜 흐름과 준비 시간

초기 구조는:

```text
Idle → Preparing → Running
```

이었으나, 설명을 읽는 시간까지 타이머가 흐르면 불공정하고, UI를 열면 시간이 멈추면 긴장감이 사라지는 문제가 있었다.

현재 구조:

```text
Idle
→ Briefing
→ Preparing
→ Running
→ Completed
```

### Briefing

- 시간 제한 없음
- Simulation pause
- 목표, 예상 위협, 보상 확인
- 연구 구매 불가
- Trait 변경 불가
- FEED/PURGE 불가

즉 **읽고 이해하는 시간**이다.

### Preparing

- 제한시간
- Simulation Time 진행
- 세포 이동/증식/대사 진행
- ENERGY 생산 진행
- 연구 구매 가능
- Trait Loadout 변경 가능
- FEED/PURGE 가능
- RESEARCH/MAIL/ANALYSIS/SYSTEM 화면을 열어도 준비 타이머는 멈추지 않음
- 일반 수동 pause 불가
- OS background/system suspend만 시간을 멈춤

즉 **결정하고 준비하는 시간**이다.

목표 감정:

> 무엇이 오는지는 이해했다. 이제 제한시간 안에 준비해야 한다.

---

# 15. Preparing에서 실제로 하는 일

중요한 정리:

“세포 수 증가”, “군집 정비”, “에너지 저장”은 별도 버튼명이 아니라 플레이어 행동의 **결과 상태**다.

Preparing 중 직접 행동은 주로:

- 연구 구매
- Trait Loadout 변경
- FEED
- 필요 시 PURGE
- 실험 조기 시작

이다.

그 결과:

- 세포 수
- 세포 내부 에너지
- ENERGY
- 영양 비축량
- 세포 위치
- 세포 밀도
- 활성 Trait

이 달라진다.

### 세포 수 증가

FEED와 시간을 통해 분열시킨다.

장점:

- 전멸에 대한 완충
- 전체 대사량 증가
- ENERGY 생산 증가
- 개체 수 목표 대응

대가:

- 영양 소비 증가
- 밀도 증가
- 자원 경쟁 증가

### 연구 구매

ENERGY/DATA를 사용한다.

효과:

- 새로운 대응 행동 해금

대가:

- 실험 시작 시 남는 ENERGY 감소

### ENERGY 비축

일부러 연구를 덜 사고 ENERGY를 남긴다.

효과:

- Running 중 PURGE 등 긴급 개입 여유

대가:

- 생명체 자체의 능력은 덜 발전

따라서:

> 세포 자체를 강하게 만들 것인가 / 내가 개입할 자원을 남길 것인가

라는 선택이 생긴다.

### Trait Loadout

Running 진입 순간 잠긴다.

즉 Preparing에서 정하는 Loadout이 이번 실험의 “빌드”다.

### FEED 위치

직접적인 효과:

- 세포 내부 에너지
- 증식
- 세포 위치

Chemotaxis 이후에는 먹이 위치로 군집 이동을 유도할 수 있다.

### 세포 위치·밀도

MVP에는 정식 Cluster 편집 기능이 없다.

따라서 “군집 형태 정비”보다 **세포 위치·밀도 정리**라고 표현하는 것이 정확하다.

FEED와 Adhesion을 통해 간접적으로 만든다.

---

# 16. Preparing → Running 상태 인계

이 부분은 직전 논의에서 권장안으로 제시되었고, 아직 정본 규칙에 최종 반영하기 전 확인 단계다.

권장 원칙:

> Preparing에서 만든 상태는 Running으로 그대로 이어지고, 해당 Protocol이 명시적으로 초기화한다고 적은 항목만 예외로 한다.

기본적으로 이어져야 할 값:

- 세포 수
- 세포 위치
- 세포 내부 에너지
- ENERGY
- 영양장
- 독성장
- Trait Loadout
- Cooldown
- 세포 상태

이 규칙이 중요한 이유는 준비 단계의 행동이 실제 실험에 직접 연결되어야 하기 때문이다.

프로토콜 시작 시 많은 상태를 자동 초기화하면 Preparing의 의미가 약해진다.

**현재 상태: 사용자에게 권장했고 논리적으로 합의 방향은 긍정적이나, CORE_GAME_RULES에 최종 문장으로 확정하기 전 재확인 필요.**

---

# 17. 저장 UX

모바일 싱글플레이에서 복잡한 슬롯 선택 화면은 피한다.

현재 구조:

### Campaign Save

- 활성 캠페인 1개
- 타이틀의 `CONTINUE` 대상
- `NEW GAME`은 기존 진행 덮어쓰기 확인

### Rolling Autosave

- 최근 정상본 3세대
- 최신 저장 손상 시 이전 세대로 fallback
- 초기 제안 주기: 약 30초 simulation time
- Protocol 완료
- 중요한 Transaction 확정 후
- app background 직전

등에서 autosave 요청

### Protocol Checkpoint

- Briefing → Preparing 진입 때 생성
- 일반 save와 별도
- 이전 Protocol checkpoint도 보존
- SYSTEM → RECOVERY에서 접근

과거 checkpoint를 복원하면 이후 autosave/checkpoint를 폐기해 서로 다른 시간선 상태가 섞이지 않게 한다.

### ProfileData

- 엔딩 기록
- 모드 해금

캠페인 rollback과 무관하게 유지한다.

일반 사용자에게 보이는 저장 UX는 최대한 단순하게:

```text
NEW GAME
CONTINUE

SYSTEM
- SAVE NOW
- RECOVERY
```

---

# 18. 한국어 현지화

사용자 결정:

> 한국어 버전에서는 게임 안에 실제로 보이는 내용이 한국어로 표시되는 것이 맞다.

현재 추가한 파일:

- `Game/Locale/ko.reference.json`
- `Game/Locale/README.md`
- `Game/Docs/current/LOCALIZATION_KO_REFERENCE.md`

`ko.reference.json`은 런타임 파일이 아니라 **개발 전 용어 검토용 구조화 데이터**다.

실제 개발 시:

```text
Locale/en.json
Locale/ko.json
```

을 동일 key 구조로 만든다.

한국어판에서 번역하는 것:

- 메뉴
- HUD
- 연구/특성명
- 효과 설명
- Protocol 제목과 목표
- 시스템 메시지
- Mail
- UNKNOWN 메시지

번역하지 않는 것:

- `Dot.exe`
- `P-01` 같은 프로토콜 ID
- `B-14` 같은 균주 ID
- 의도적으로 깨진 데이터/바이너리

UNKNOWN 초기 통신은 정상적인 문장으로 번역하지 않는다.

예:

```text
YOU
→ 너

YOU CHANGE
→ 너 바꾼다

YOU CHANGE WORLD
→ 너 세상 바꾼다
```

언어를 배우는 과정의 어색함을 유지하기 위해서다.

---

# 19. UI 시안 진행 기록

## 19.1 첫 UI 3안

`Game/Design/UI/Dotexe_UI_Concepts.html`

세 방향을 만들었다.

### A. 관찰 콘솔

- 배양 화면 최우선
- UI 최소
- 세포 관찰성 강조

### B. 계측 장비 패널

- 연구 장비 판타지 강조
- 수치 판독성
- 패널 구조

### C. 암흑 터미널

- 공포/서사 분위기 강조
- 화면 전체를 배양 공간처럼 사용
- 로그와 최소 HUD

사용자는 **B 계측 장비 패널**을 가장 선호했다.

---

# 20. B안 수정 3종

`Game/Design/UI/Dotexe_UI_B_Revisions.html`

사용자 피드백:

- 기존 폰트가 충분히 도트 느낌이 아님
- 둥근 세포가 현대적으로 보임
- 첨부 레퍼런스처럼 더 어둡고 공포스럽게
- 디지털 지직 연출 필요
- 패널/버튼은 더 미니멀하게
- 세포 수/ENERGY 등을 게임 재화처럼 읽히게

이에 B안을 세 가지로 다시 만들었다.

### B-1 Silent Instrument

- 첨부 레퍼런스에 가장 가까움
- 큰 배양 공간
- 얇은 계측선
- UI 최소화
- 상단 자원 아이콘 + 숫자

### B-2 Modular Rack

- 게임성과 자원 판독성 강화
- 에너지/세포/영양을 작은 장비 모듈처럼 표시

### B-3 Distorted Diagnostic

- 공포·진단 로그 강화
- 검은 화면
- 최소 수치
- 간헐 글리치

사용자의 최신 선택은 **B-1 Silent Instrument**다.

---

# 21. 최신 B-1 수정 방향

B-1 선택 이후 사용자가 추가로 요청한 사항이다.

이 부분은 **현재 가장 최신 UI 기준**이다.

## 21.1 프로토콜 표시

기존 좌측 상단의 “관찰 중”을 제거한다.

대신:

```text
프로토콜 P-02 00:43 ▼
```

처럼 표시한다.

화살표를 누르면 세부 내용이 아래로 펼쳐진다.

세부 정보 예:

- 목표
- 예상 위협
- 필요 개체 수
- 실험까지 남은 시간
- 필요 시 권장 준비

접으면 한 줄로 돌아간다.

## 21.2 세포와 영양 픽셀

최종 요청:

- **세포 = 하얀색 정사각형 픽셀 1칸**
- **영양분 = 초록색 정사각형 픽셀 1칸**
- 영양분은 세포보다 더 작음

중요:

> 모양만 픽셀이지 이동은 픽셀 단위가 아니다.

세포는 연속 좌표에서 자유롭게 움직이며 화면에서도 부드럽게 이동해야 한다.

## 21.3 CRT

기존보다:

- 가장자리 곡률 조금 강화
- 가장자리 왜곡 조금 강화
- 낡은 화면 같은 비네팅 추가
- 전체 화면 밝기는 소폭 상향

방해되지 않는 것이 최우선이다.

## 21.4 현재 구현 상태

이 최신 B-1 수정안을 반영한 HTML은 채팅 sandbox에서 한 차례 생성되었다.

예정 GitHub 경로:

`Game/Design/UI/Dotexe_UI_B1_refined.html`

그러나 **이 맥락 문서 작성 요청이 들어오면서 GitHub 업로드 직전에 작업이 중단되었다.**

따라서 다음 UI 작업을 재개할 때 가장 먼저 할 일:

1. 최신 B-1 HTML을 다시 확인
2. `Game/Design/UI/Dotexe_UI_B1_refined.html`로 commit
3. PR #1에 연결
4. 사용자가 브라우저에서 확인
5. 피드백 후 최종 UI 방향 확정

이 상태를 “이미 GitHub에 올라갔다”고 가정하면 안 된다.

---

# 22. UI에서 현재 중요하게 보는 판단 기준

최종 UI를 평가할 때 다음을 본다.

1. 세포를 10초 이상 가만히 보고 싶어지는가
2. 관찰 화면이 UI에 눌리지 않는가
3. ENERGY / 세포 수 / 영양 비축량이 한눈에 게임 자원처럼 읽히는가
4. 연구 버튼을 눌렀을 때 연구 장비를 조작하는 느낌이 나는가
5. 준비 타이머가 긴장을 주되 정보를 읽지 못하게 하지 않는가
6. 한 손 터치에 무리가 없는가
7. CRT 효과를 줄여도 상태 판독성이 유지되는가
8. 후반 UNKNOWN/글리치 연출이 들어갈 공간이 남아 있는가
9. 현재 화면이 너무 현대적인 앱 UI처럼 보이지 않는가
10. 픽셀 표현과 자유 이동이 동시에 자연스럽게 느껴지는가

---

# 23. Mail / Emergence / 스토리 원칙

스토리는 메일만으로 진행하지 않는다.

반드시:

```text
실제 행동
→ Observation Record
→ Narrative Condition
→ Mail
```

순서다.

메일이 먼저 “세포가 예측했다”고 주장하면 안 된다.

화면에서 실제 예측 행동이 발생하고 기록된 뒤에 메일이 와야 한다.

Emergence는 IQ 게이지 같은 단순 점수로 표현하지 않는다.

Evidence 종류:

- Memory
- Coordination
- Prediction
- Autonomy

발견은 한 번만 기록되는 행동 증거다.

플레이어가 어떤 연구 경로를 선택하더라도 높은 단계의 서사에 도달할 수 있어야 한다.

특정 정답 연구 하나를 강제하지 않는다.

---

# 24. 현재 큰 스토리 구조

현재 뼈대:

### Phase 1 — Culture

정상적인 세포 배양으로 보임.

### Phase 2 — Adaptation

감염 실험. 일부 세포가 자극 전에 움직인 기록이 보이기 시작.

### Phase 3 — Pattern

설명하기 힘든 구조와 반복 행동.

로그에 다음 단어가 섞이기 시작:

- SIMULATION STABILITY
- COMPUTE BUDGET
- STATE RESTORE

### Phase 4 — Contact

UNKNOWN 데이터.

```text
YOU
→ YOU CHANGE
→ YOU CHANGE WORLD
```

### Phase 5 — Realization

세포가 배양 공간의 “경계”를 탐색한다.

단순 벽 충돌이 아니라:

- 여러 경계 반복 탐색
- 경계선을 따라 정렬
- 경계 주변 신호 교환

같은 행동 후:

```text
WE FOUND THE BORDER.
```

가 나온다.

마지막에는 플레이어보다 먼저 반응하는 행동이 나타난다.

---

# 25. 최종 프로토콜

최종 실험은 플레이어의 수동 조작을 잠근다.

금지:

- FEED
- SIGNAL
- PURGE
- ISOLATE
- ENVIRONMENT
- Trait 변경

목표는 지금까지 플레이어가 만들어 온 생명체가 **운용자 없이 스스로 살아남고 대응할 수 있는가**다.

단계:

```text
EQUILIBRIUM
→ DISTURBANCE
→ RECOVERY
```

성공에는 단순 생존뿐 아니라:

- 전멸하지 않음
- 최종 개체 조건
- 오염 조건
- Autonomous Response 관찰
- 회복 추세

가 필요하다.

---

# 26. 엔딩 방향

현재 3개 선택:

- SHUT DOWN
- MAINTAIN SYSTEM
- OPEN CHANNEL

정답 엔딩은 없다.

최종 UNKNOWN의 핵심 질문:

```text
이곳에는 규칙이 있다.
에너지.
시간.
죽음.

너는 그것들을 바꿨다.

네 세계에도 규칙이 있는가?
네 규칙은 누가 바꾸는가?
```

플레이어가 세포에게 “세계 바깥 존재”였듯, 플레이어 자신의 세계에도 같은 구조가 있을 수 있다는 여운을 남기는 것이 목적이다.

---

# 27. MVP 범위

MVP는 전체 스토리를 만드는 단계가 아니다.

검증 질문:

1. 한 점이 살아 움직이는 느낌이 있는가
2. 세포 수가 늘어날수록 보는 재미가 커지는가
3. 먹이 위치 조작이 실제 플레이처럼 느껴지는가
4. 연구가 행동 변화를 만든다고 느끼는가
5. 다음 프로토콜 준비가 전략적 선택을 만드는가
6. Rapid Bacteria와의 첫 방어가 재미있는가

MVP:

- 최대 300 friendly cells
- Nutrient / Toxin / Alarm Signal field
- FEED
- PURGE
- ENERGY / DATA / NUTRIENT RESERVE
- 연구 10개
- 글로벌 Trait Loadout
- Rapid Bacteria
- Protocol P-01~P-03
- Phase 1~2 수준 Mail 8~10개

MVP 이후:

- Mutation
- 개별 세포 Trait
- 역할 분화
- Environment
- Flow
- Waste / Contamination
- 정식 Cluster
- 플레이어 SIGNAL
- ISOLATE
- 능동 Analysis
- Virus / Parasite / Fungus
- 전체 Emergence
- UNKNOWN 전체 시퀀스
- Final Protocol

---

# 28. 실개발 전에 아직 닫아야 할 것

사용자 의도상 다음 큰 작업은 **스토리 분야를 먼저 닫는 것**이다.

즉 다음 순서가 적절하다.

### 1. UI B-1 최종안 확정

최신 B-1 수정안 검토 후:

- HUD
- 프로토콜 펼침
- 연구 화면
- Mail
- Analysis
- System
- Briefing
- Game Over

까지 동일 디자인 언어로 확장한다.

### 2. MVP 이후 캠페인 전체

Protocol을 처음부터 끝까지 정의한다.

각 Protocol마다:

- 등장 시점
- Briefing
- 환경/적
- 목표
- 준비 시간
- 보상
- 가능한 대응 경로
- Observation
- Evidence
- Mail
- 다음 해금

을 연결한다.

### 3. 모든 Mail 작성

Miller / J. Park / SYSTEM / UNKNOWN의 전체 메일을 순서와 조건까지 확정한다.

### 4. Emergence 그래프 확정

```text
Research
→ 실제 행동
→ Observation
→ Evidence
→ Narrative Gate
→ Mail
→ 다음 Protocol / Feature
```

전체 캠페인에 대해 매핑한다.

### 5. 밸런스 데이터

실개발 전 또는 각 milestone 진입 전에:

- 연구 비용
- DATA 비용
- Protocol 목표 수치
- Preparation 시간
- Rapid Bacteria 스탯
- PURGE 비용/범위/피해/cooldown
- FEED 양
- field 확산/감쇠
- autosave 간격
- fixed-step catch-up

등을 데이터로 확정한다.

---

# 29. 현재 미확정 또는 재검토할 항목

다음은 아직 “완전 확정”으로 간주하면 안 된다.

### 한국어 연구명

전체 초안은 존재하지만 사용자 최종 검토 전.

### Preparing 상태 인계

Preparing의 모든 시뮬레이션 상태를 Running으로 그대로 넘기는 것을 권장했으나, CORE_GAME_RULES에 최종 확정 전.

### Low-Energy Division

“더 적은 에너지로 분열 / 새 세포 초기 에너지 감소”의 에너지 보존 규칙을 구현 전 더 명확히 해야 한다.

### Alarm Signal / Threat Signal

MVP의 Alarm Signal과 후반 Signaling 계통 Threat Signal의 역할이 겹치지 않도록 캠페인 설계 때 분리 필요.

### Final Protocol Recovery 판정

“회복 추세”의 실제 수학적 판정식은 M7 전에 확정 필요.

### 전체 캠페인 Emergence 경로

원칙은 확정되어 있으나 실제 Protocol별 Evidence 매트릭스는 아직 미작성.

### 게임 라이선스

Luna 라이선스와 게임 프로젝트를 분리했지만 Dot.exe 자체의 공개/비공개 라이선스 정책은 아직 결정하지 않음.

---

# 30. 개발 시 작업 원칙

- 규칙이 비어 있으면 코드에서 임의 결정하지 않는다.
- 먼저 `CORE_GAME_RULES.md`를 수정한다.
- UI가 게임 상태를 직접 바꾸지 않는다.
- Command / Transaction → Service를 거친다.
- Renderer에 규칙을 넣지 않는다.
- Balance 값은 data로 분리한다.
- SaveData 변경 시 migration과 test를 추가한다.
- Gameplay RNG와 Presentation RNG를 분리한다.
- feature 간 직접 의존을 최소화한다.
- 시뮬레이션은 화면 없이 headless 실행 가능해야 한다.
- Fixed tick + 가변 render 구조를 유지한다.
- 공간 검색은 Spatial Hash 사용, O(n²) 근접 검색 금지.
- 모바일 세로 1080×1920 / 논리 216×384 기준을 유지한다.
- unrelated refactor를 하지 않는다.

---

# 31. 테스트 철학

각 개발 단계는:

```text
에이전트 헤드리스 배속 테스트
→ 브라우저 실제 화면 배속 테스트
→ 사람 테스트
```

순서다.

재미/감정은 자동 테스트가 판정하지 않는다.

예:

- 세포를 보고 싶어지는가
- Chemotaxis 전후 변화가 짜릿한가
- 준비 시간이 긴장되는가
- Rapid Bacteria가 불공정하지 않은가
- UNKNOWN 등장 전부터 이상함이 서서히 쌓이는가

이런 질문은 사람이 직접 확인해야 한다.

---

# 32. 현재 GitHub 작업 흐름

현재 작업은 `docs/project-normalization` 브랜치와 PR #1에서 이어지고 있다.

PR에는 지금까지:

- 프로젝트 구조 분리
- 문서 current/history 분리
- Dot.exe 명칭 통일
- ENERGY 명칭 통일
- 세로형 ResponsiveShell 규칙
- P-01/prototype 일치
- 저장 UX
- Briefing/Preparing 구조
- 한국어 번역 reference
- UI 시안
- B안 수정 UI 3종

이 들어 있다.

새 작업 시 main에 곧바로 덮어쓰기보다 이 브랜치/PR의 최신 상태를 먼저 확인한다.

---

# 33. 이 맥락 파일의 유지 규칙

이 파일은 앞으로 지속적으로 갱신한다.

다음과 같은 변경이 생기면 해당 작업과 같은 커밋 또는 바로 다음 커밋에서 이 파일을 수정한다.

- 게임의 핵심 방향 변경
- 시스템 이름 변경
- UI 최종안 선택
- Protocol 구조 변경
- 자원 추가/삭제
- 저장 UX 변경
- 캠페인 구조 확정
- 주요 스토리 반전 변경
- 새로운 개발 단계 진입
- 중요한 기술 선택 변경
- “왜 이렇게 했는지”를 모르면 이후 판단이 달라질 수 있는 결정

반대로 숫자 하나의 미세한 밸런스 조정이나 단순 오탈자 수정은 이 파일에 모두 기록하지 않는다.

목표는 이 파일을 읽은 새 채팅/새 에이전트가:

1. Dot.exe가 어떤 게임인지
2. 현재 무엇이 확정됐는지
3. 무엇이 아직 열려 있는지
4. 지금 가장 먼저 해야 할 일이 무엇인지
5. 과거에 왜 특정 선택을 했는지

를 채팅 기록 없이 복구할 수 있게 하는 것이다.

---

# 34. 다음 재개 지점

**가장 가까운 미완료 작업:**

`Game/Design/UI/Dotexe_UI_B1_refined.html`을 최신 B-1 요구사항으로 GitHub에 추가하고 사용자 검토를 받는다.

최신 요구사항:

```text
좌측 상단:
프로토콜 P-02 00:43 ▼
→ 터치 시 상세 펼침

세포:
흰색 정사각 픽셀
부드러운 자유 이동

영양:
세포보다 작은 초록색 정사각 픽셀

CRT:
곡률 ↑
가장자리 왜곡 ↑
비네팅 추가
전체 밝기 소폭 ↑
지직 연출은 짧고 드물게
```

그 UI가 승인되면 같은 디자인 언어로 나머지 게임 화면을 확장한다.

그 다음 큰 작업은 사용자가 이미 예고한 대로:

**MVP 이후 스토리 / 캠페인 / 전체 시스템 메일 / Emergence 경로를 함께 닫는 것**이다.
