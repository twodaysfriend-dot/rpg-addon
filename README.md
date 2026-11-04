# Minecraft Bedrock RPG 애드온 v0.5.32

[설치용 애드온 다운로드](downloads/RPG_v0532.mcaddon?raw=true) · [소스와 안내 ZIP](downloads/RPG_v0532_source_and_notes.zip?raw=true)

파일 페이지가 열리면 Download raw file 버튼으로 내려받으세요. `.mcaddon`을 Minecraft로 열어 가져오고 월드에 행동팩·명령어팩·리소스팩을 적용하세요. 기존 UUID를 유지하는 업데이트이므로 먼저 월드를 복사하고 세 팩의 버전이 모두 0.5.32인지 확인하세요.

이번 변경은 히드라 XP 900·마공학 골렘 XP 850, 건초 9개 샌드백 제작, 성직자 광역 지원 아군 한정, 카멜레온 기술 호환, 재접속 쿨타임 유지, 생명 룬 최대 HP +24, 지능 마나 절약 최대 40%입니다. 분해 후 부여된 버프를 제거하는 처리는 추가하지 않았습니다.

- [최신 변경과 API 제한](docs/v0532-changes.md)
- [풀강 비용·시간 비교](docs/v0532-enhancement-costs.md)
- [직업 계열별 129개 스킬](docs/v0532-skills.md)
- [계산 원본 수치](docs/v0532-enhancement-costs.json)
- [파일 SHA-256](downloads/SHA256SUMS_v0532.txt)

검증: 모의 API JavaScript 시나리오 135개와 Python 검증 5개 통과. JSON 433개·JS 114개 구문·활성 모듈 경로·팩 의존성·ZIP 무결성 검사 완료. 실제 Bedrock 클라이언트 실기 테스트는 수행하지 못했습니다. 일반 공격 전역 아군 피해 차단과 레벨 방어 등 일부 피해 전 이벤트 기능에는 API 제한이 남아 있습니다.

이전 배포: [v0.5.31](downloads/RPG_v0531.mcaddon?raw=true).
