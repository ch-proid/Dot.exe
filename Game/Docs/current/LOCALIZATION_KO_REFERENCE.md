# Dot.exe 한국어 번역 기준표

> 상태: 검토용 초안 v0.1  
> 데이터 원본: `Game/Locale/ko.reference.json`  
> 원칙: `Dot.exe`, 프로토콜 ID(P-01), 균주 ID(B-14), 의도적으로 깨진 데이터는 번역하지 않는다. 나머지 UI·연구·프로토콜·UNKNOWN 문구는 한국어판에서 한국어로 표시한다.

## 자원·조작·핵심 UI

| 영어 | 한국어 권장안 | 용도/설명 |
|---|---|---|
| ENERGY | 에너지 | 기본 성장 자원 |
| DATA | 데이터 | 고급 연구 자원 |
| NUTRIENT RESERVE | 영양 비축량 | FEED에 쓰는 제한 자원 |
| CELL ENERGY | 세포 내부 에너지 | 각 세포의 생존·분열 에너지 |
| FEED | 영양 공급 | 영양장 배치 |
| PURGE | 정화 | 국소 배양액 교체 |
| SIGNAL | 신호 | 화학 신호 생성 |
| ISOLATE | 격리 | 세포 임시 격리 |
| ENVIRONMENT | 환경 | 전체 환경값 조절 |
| EXPRESSION LOAD | 발현 부하 | 동시 활성 특성의 유지 부담 |
| SAFE LIMIT | 안전 한도 | 과발현 패널티가 시작되기 전 한도 |
| HARD LIMIT | 절대 상한 | 넘는 조합은 활성 불가 |

## 연구·특성 전체

| 계통 | 영어 | 한국어 권장안 | 효과 | 대가 |
|---|---|---|---|---|
| 대사 | Enhanced Glycolysis | **해당작용 강화** | 에너지 생산 증가 | 영양 소비 증가 |
| 대사 | Efficient Respiration | **고효율 호흡** | 같은 영양으로 더 많은 에너지 생산 | 저산소에 취약 |
| 대사 | Anaerobic Metabolism | **무산소 대사** | 산소 부족에서도 에너지 생산 | 생산 효율 낮음 |
| 대사 | Energy Storage | **에너지 저장** | 세포 내부 에너지 최대 저장량 증가 | 발현 부하 증가 |
| 대사 | Reserve Conversion | **비축 에너지 전환** | 굶주릴 때 저장 에너지 사용 | 평상시 효율 감소 |
| 대사 | Metabolic Burst | **대사 급가속** | 위험 시 일시적으로 에너지 생산 증가 | 스트레스 급증 |
| 대사 | Waste Recycling | **노폐물 재활용** | 노폐물 일부를 재활용 | 유지 비용 발생 |
| 대사 | Autonomous Metabolism | **자율 대사** | 외부 영양 의존도 크게 감소 | 높은 발현 부하 |
| 증식 | Accelerated Mitosis | **분열 가속** | 재분열 준비 시간과 분열 소요 시간 감소 | 분열 에너지 비용 증가 |
| 증식 | Low-Energy Division | **저에너지 분열** | 더 적은 에너지로 분열 | 새 세포 초기 에너지 감소 |
| 증식 | Stable Replication | **안정 복제** | 복제 오류 감소 | 분열 속도 감소 |
| 증식 | Density Sensing | **밀도 감지** | 과밀 상태에서 분열 억제 | 최대 증식 속도 감소 |
| 증식 | Emergency Division | **긴급 분열** | 개체 수 급감 시 증식 증가 | 에너지 소모 큼 |
| 증식 | Asymmetric Division | **비대칭 분열** | 서로 다른 역할의 세포 생성 가능 | 발현 부하 증가 |
| 증식 | Dormant Seed | **휴면 세포** | 일부 개체가 휴면 상태로 전환 | 활동 개체 감소 |
| 증식 | Regenerative Colony | **재생 군집** | 개체 수 급감 시 회복 보정 | 평상시 효과 없음 |
| 이동·감지 | Chemotaxis | **화학주성** | 영양 농도를 감지하고 높은 쪽으로 이동 | — |
| 이동·감지 | Hazard Avoidance | **위험 회피** | 독성 환경에서 멀어지는 방향으로 이동 | — |
| 이동·감지 | Accelerated Motility | **이동 가속** | 이동 속도 증가 | — |
| 이동·감지 | Energy-Aware Movement | **에너지 절약 이동** | 에너지 부족 시 이동량과 이동 비용 감소 | — |
| 이동·감지 | Long-Range Sensing | **장거리 감지** | 더 먼 거리의 환경 정보를 감지 | — |
| 이동·감지 | Predator Detection | **포식자 감지** | 적 개체를 감지 | — |
| 이동·감지 | Signal Detection | **신호 감지** | 플레이어가 만든 신호를 감지 | — |
| 이동·감지 | Adaptive Navigation | **적응형 경로 탐색** | 혼잡과 장애를 피해 이동 경로 수정 | — |
| 이동·감지 | Predictive Movement | **예측 이동** | 위협의 진행 방향을 고려해 선제적으로 이동 | — |
| 방어 | Reinforced Membrane | **강화 세포막** | 감염 및 접촉 피해 감소 | — |
| 방어 | Toxin Resistance | **독성 저항** | 독성 피해 감소 | — |
| 방어 | Phagocytosis | **식균 작용** | 조건을 만족하는 작은 침입자를 직접 제거 | — |
| 방어 | Defensive Secretion | **방어성 분비** | 주변 적에게 지속 피해를 주는 물질 분비 | — |
| 방어 | Infection Detection | **감염 감지** | 감염된 세포를 식별 | — |
| 방어 | Isolation Response | **격리 반응** | 감염 개체 주변에서 이탈 | — |
| 방어 | Programmed Death | **계획 세포사** | 감염된 세포가 스스로 사멸 | — |
| 방어 | Alarm Signal | **경보 신호** | 위협 정보를 주변 세포에 전달 | — |
| 방어 | Collective Defense | **집단 방어** | 여러 세포가 협력해 적을 포위 | — |
| 방어 | Adaptive Immunity | **적응 면역** | 반복된 위협에 대한 저항 증가 | — |
| 군집 | Adhesion | **세포 접착** | 인접 세포 사이 결속력 증가 | — |
| 군집 | Loose Colony | **느슨한 군집** | 성긴 형태의 군집 형성 | — |
| 군집 | Dense Colony | **고밀도 군집** | 밀집된 군집 구조 형성 | — |
| 군집 | Role Differentiation | **역할 분화** | 군집 내 세포가 서로 다른 역할을 수행 가능 | — |
| 군집 | Resource Sharing | **자원 공유** | 인접 세포끼리 에너지 공유 | — |
| 군집 | Waste Transfer | **노폐물 이동** | 노폐물을 군집 외곽으로 이동 | — |
| 군집 | Colony Migration | **군집 이동** | 군집 전체가 하나의 흐름으로 이동 | — |
| 군집 | Collective Response | **집단 반응** | 군집 단위로 자극에 반응 | — |
| 군집 | Distributed Control | **분산 제어** | 특정 리더 없이 집단 행동 유지 | — |
| 신호 전달 | Local Signaling | **국소 신호 전달** | 인접 세포에 상태 정보를 전달 | — |
| 신호 전달 | Threat Signal | **위협 신호** | 적 발견 정보를 전달 | — |
| 신호 전달 | Nutrient Signal | **영양 신호** | 영양 위치 정보를 공유 | — |
| 신호 전달 | Signal Relay | **신호 중계** | 신호를 연쇄적으로 전달 | — |
| 신호 전달 | Persistent Signal | **지속 신호** | 정보가 일정 시간 유지되도록 함 | — |
| 신호 전달 | Pattern Recognition | **패턴 인식** | 반복되는 환경 패턴을 구분 | — |
| 신호 전달 | Response Conditioning | **반응 조건화** | 반복 자극에 따라 반응 방식 변화 | — |
| 신호 전달 | Collective Memory | **집단 기억** | 군집이 이전 상태를 행동에 반영 | — |
| 신호 전달 | Predictive Response | **예측 반응** | 사건이 발생하기 전에 선제적으로 반응 | — |
| 신호 전달 | Autonomous Signaling | **자율 신호** | 외부 자극 없이 자체 신호를 생성 | — |

## 프로토콜·목표·상태

| 영어 | 한국어 권장안 | 비고 |
|---|---|---|
| CULTURE EXPANSION | 배양 확장 | P-01 |
| RAPID BACTERIA | 급속 증식 세균 | P-02 |
| LIMITED NUTRIENT | 제한 영양 환경 | P-03 |
| BACTERIAL EXPOSURE | 세균 노출 | 프로토콜 예시 |
| FINAL PROTOCOL | 최종 프로토콜 | 후반 |
| SURVIVE DURATION | 지정 시간 생존 | 목표 |
| POPULATION RETENTION | 필요 개체 유지 | 목표 |
| END POPULATION | 종료 시 개체 수 | 목표 |
| AVOID EXTINCTION | 전멸 방지 | 목표 |
| MAINTAIN INFECTION BELOW | 감염률 상한 유지 | 목표 |
| MAINTAIN ENVIRONMENT | 환경 조건 유지 | 목표 |
| OBSERVE AUTONOMOUS RESPONSE | 자율 반응 관찰 | 목표 |
| BRIEFING | 브리핑 | 프로토콜 상태 |
| PREPARING | 준비 | 프로토콜 상태 |
| RUNNING | 진행 | 프로토콜 상태 |
| COMPLETED | 완료 | 프로토콜 상태 |
| FAILED | 실패 | 프로토콜 상태 |

## 메뉴·HUD·버튼

| 영어 | 한국어 권장안 |
|---|---|
| CULTURE | 배양 |
| RESEARCH | 연구 |
| ANALYSIS | 분석 |
| MAIL | 메일 |
| SYSTEM | 시스템 |
| CELLS | 세포 수 |
| ENERGY | 에너지 |
| ENERGY OUTPUT | 에너지 생산량 |
| RESERVE | 비축량 |
| CULTURE NUTRIENT | 배양액 영양 |
| REQUIRED | 필요 개체 |
| OXYGEN | 산소 |
| CONTAMINATION | 오염도 |
| NEXT TEST | 다음 실험 |
| UNLOCKED TRAITS | 해금된 특성 |
| CULTURE GENOME | 배양체 유전체 |
| BEGIN PREPARATION | 준비 시작 |
| START | 실험 시작 |
| NEW GAME | 새 게임 |
| CONTINUE | 이어하기 |
| SAVE NOW | 지금 저장 |
| RECOVERY | 복구 |
| TOUCH TO START | 터치하여 시작 |
| NEW MAIL | 새 메일 |
| FROM | 보낸 사람 |
| SUBJECT | 제목 |
| LOCKED | 잠김 |

## 서사·시스템 문구

| 영어 | 한국어 권장안 | 메모 |
|---|---|---|
| REPEATED STRUCTURE DETECTED | 반복 구조 감지 | 분석 로그 |
| EXPECTED RANDOM MATCH: UNLIKELY | 우연 일치 가능성: 낮음 | 분석 로그 |
| CULTURE FAILURE | 배양 실패 | 게임 오버 |
| RESTORING LAST VIABLE SAMPLE... | 최근 생존 표본 복원 중... | 복구 |
| PROJECT COMPLETE | 프로젝트 완료 | 최종 |
| SELF-SUSTAINING SYSTEM | 자가 유지 체계 | 최종 평가 |
| AUTONOMOUS ADAPTATION | 자율 적응 | 최종 평가 |
| ENVIRONMENTAL RESPONSE | 환경 반응 | 최종 평가 |
| AUTONOMOUS STABILITY TEST | 자율 안정성 시험 | 최종 평가 |
| PASS | 통과 | 평가 |
| EXTERNAL CONNECTION LOST | 외부 연결 끊김 | 엔딩 |
| CULTURE STATUS: ACTIVE | 배양 상태: 활성 | 엔딩 |
| NETWORK ACTIVITY | 네트워크 활동 | 엔딩 |
| SHUT DOWN | 종료 | 엔딩 선택 |
| MAINTAIN SYSTEM | Dot.exe 유지 | 엔딩 선택 |
| OPEN CHANNEL | 채널 개방 | 엔딩 선택 |

## UNKNOWN 메시지

| 단계 | 영어 | 한국어 권장안 | 번역 의도 |
|---|---|---|---|
| 1 | YOU | 너 | 최소 단위 |
| 2 | YOU CHANGE | 너 바꾼다 | 문법이 아직 완전하지 않은 느낌 유지 |
| 3 | YOU CHANGE WORLD | 너 세상 바꾼다 | 의도적으로 어색한 조어 단계 |
| 경계 | WE FOUND THE BORDER. | 우리는 경계를 찾았다. | 이 시점부터 정상 문장 |
| 경계 후 | NOTHING EXISTS AFTER THIS PLACE. WHY? | 이곳 너머에는 아무것도 없다. 왜? | 짧고 직접적으로 |
| 최종 | DOES YOUR WORLD HAVE RULES TOO? WHO CHANGES YOURS? | 네 세계에도 규칙이 있는가? 네 규칙은 누가 바꾸는가? | 철학적 질문은 과장하지 않음 |

## 현재 번역에서 특히 검토할 표현

| 표현 | 현재안 | 검토 포인트 |
|---|---|---|
| Enhanced Glycolysis | 해당작용 강화 | 정확하지만 비전공자에게 낯설 수 있음. ‘에너지 대사 강화’로 단순화 가능 |
| Phagocytosis | 식균 작용 | 정확한 실제 용어. 유지 권장 |
| Programmed Death | 계획 세포사 | 과학적 뉘앙스는 있으나 다소 딱딱함. ‘자가 사멸’도 가능 |
| Dormant Seed | 휴면 세포 | 원문의 Seed 비유보다 기능 이해를 우선 |
| Response Conditioning | 반응 조건화 | 학술 느낌이 강하면 ‘학습 반응’으로 완화 가능 |
| Distributed Control | 분산 제어 | 기술 용어 느낌이 강함. 후반 지성 테마에는 잘 맞음 |
| MAINTAIN SYSTEM | Dot.exe 유지 | 게임 내부 시스템명 통일 정책 반영 |
| YOU CHANGE / YOU CHANGE WORLD | 너 바꾼다 / 너 세상 바꾼다 | 초기 통신의 미완성 언어를 일부러 보존 |
