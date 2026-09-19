# Locale reference

현재는 코드 구현 전 용어 검토 단계다.

- `ko.reference.json`: 한국어판 용어·문구 검토용 구조화 데이터
- `Game/Docs/current/LOCALIZATION_KO_REFERENCE.md`: 사람이 읽고 검토하기 쉬운 표

실제 런타임 구현 시에는 이 기준을 바탕으로 `en.json`과 `ko.json`을 동일 key 구조로 생성한다. `ko.reference.json`은 런타임 로딩 파일이 아니라 기획/개발 기준 데이터다.
