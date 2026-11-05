# RPG 애드온 v0.5.62 — 공용 무기 손 표시

[설치용 애드온 다운로드](downloads/RPG_v0562.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0562_source_and_notes.zip?raw=true)

공용 무기 7종을 엔진 기본 검·도끼 표시 설정으로 통일했습니다. 기존 1인칭 0.11 크기 보정은 제거하고 BP·RP의 hand_equipped 설정을 활성화했습니다. 원본 PNG·아이템 ID·피해·내구도·강화·등급·인챈트는 유지합니다.

대상: 적월선, 마공학 검, 마공학 대검, 정예기사의 대검, 빙설의 대검, 화염의 대검, 마수의 도끼.

[히든 직업 25개 목록 — 데이터 등록 순서, 전직 방법 제외](docs/v0562-hidden-jobs.md) · [변경 안내](docs/v0562-changes.md) · [SHA-256](downloads/SHA256SUMS_v0562.txt)

v0.5.61의 전직관 소환알 18종·던전/신전 보호·실패 몹 정리를 포함합니다. BP·명령어 BP·RP를 모두 0.5.62로 갱신·활성화하세요. 기존 UUID를 유지합니다.

자동 테스트 417개 통과(JavaScript 412 + Python 5). JSON 494개·JS 구문 154개·로컬 import 582개·무기 도트 7개 원본 동일성·팩 무결성을 확인했습니다. 실제 모바일 Bedrock의 손 위치·각도·크기는 미검증입니다.

[선택 BDS API 자료](downloads/RPG_v0562_API_server.zip?raw=true). 일반 설치는 위 mcaddon 파일을 사용하세요.

이전 배포: [v0.5.61](downloads/RPG_v0561.mcaddon?raw=true).
