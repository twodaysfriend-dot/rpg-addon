# RPG 애드온 v0.5.63 — 일반 상위 직업 분류 수정

[설치용 애드온 다운로드](downloads/RPG_v0563.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0563_source_and_notes.zip?raw=true)

검성·타락검성·현자·천궁·천사·미카엘·타락천사·권골을 일반 상위 직업으로 수정했습니다. 견습 제외 일반 직업 13개, 히든 직업 17개입니다. 직업 메뉴와 전직관의 분류 표시도 수정했습니다.

[히든 직업 17개 목록 — 전직 방법 제외](docs/v0563-hidden-jobs.md) · [변경 안내](docs/v0563-changes.md) · [SHA-256](downloads/SHA256SUMS_v0563.txt)

전직 조건·직전 직업·스킬·능력치는 유지했습니다. 기본 직업 5개를 대상으로 하는 기존 진행도와 GPT NPC의 공개 전직 안내 범위도 유지합니다. 상위 일반 직업의 안내를 거절할 때 히든이라고 잘못 표현하지 않도록 문구를 수정했습니다.

공용 무기 기본 도구 손 표시·원본 도트·장비 강화/합성·전직관 소환알·구조물 보호·실패 몹 정리를 포함합니다. 동봉된 BP·명령어 BP·RP를 모두 0.5.63으로 갱신·활성화하세요. 기존 UUID와 저장 데이터를 유지합니다.

자동 테스트 417개 통과(JavaScript 412 + Python 5). 직업 데이터는 hidden 필드 외에 바뀌지 않은 것을 비교 검증했습니다. JSON 494개·JS 구문 154개·로컬 import 582개·PNG 원본 동일성·팩 무결성도 확인했습니다. 실제 모바일 Bedrock 실행은 미검증입니다.

[선택 BDS API 자료](downloads/RPG_v0563_API_server.zip?raw=true). 일반 설치는 위 mcaddon 파일을 사용하세요.

이전 배포: [v0.5.62](downloads/RPG_v0562.mcaddon?raw=true).
