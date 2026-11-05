# RPG 애드온 v0.5.60 — 원본 도트·장비 +10·합성

[설치용 애드온 다운로드](downloads/RPG_v0560.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0560_source_and_notes.zip?raw=true)

사용자가 올린 RPG_v0557_modified.mcaddon을 바탕으로 한 v0.5.59에 적용했습니다. 해당 수정본의 전직·진행 설정을 유지합니다.

- 무기 아이콘 7개는 weapon_icons_128x128.zip의 원본 PNG를 바이트 변경 없이 적용했습니다.
- 룬·RPG 갑옷·공용 무기 7종은 각각 일반~신화 등급을 가집니다. 직업 전용 시전 아이템·바닐라 장비는 제외합니다.
- `/rpg:upgrade`: 갑옷·룬·손에 든 공용 무기를 최대 +10 강화합니다. 기존 룬 +5는 효과를 유지하며 +10으로 이전됩니다.
- `/rpg:fuse`: 같은 종류·같은 등급의 +10 장비 두 개를 합성합니다. 성공하면 다음 등급 +0, 실패하면 재료만 소모하고 주장비 +10 유지.
- 승급 확률은 순서대로 90/85/80/75/70/60%. 등급별 효과는 [상세 안내](docs/v0560-changes.md)에 있습니다.

[강화 비용·시간 모델](docs/v0560-enhancement-costs.md) · [원본 도트 검증 기록](docs/v0560-original-textures.json) · [SHA-256](downloads/SHA256SUMS_v0560.txt)

일반 갑옷 4부위·룬 6종 +10 강화 재화의 약 20시간 모델을 유지하도록 룬 강화 비용과 던전 촉매 보상을 조정했습니다. 신화 합성·공용 무기까지 20시간에 완성된다는 뜻은 아닙니다.

동봉된 BP·명령어 BP·RP를 모두 0.5.60으로 갱신하고 활성화하세요. Bedrock 1.21.100 이상이 필요합니다.

자동 검증 400개 통과(JavaScript 395 + Python 5), JSON 492개·JS 구문 152개·로컬 import 577개·PNG 7개 원본 동일성·세 팩 ZIP 무결성을 확인했습니다. 실제 모바일 Bedrock 플레이 실측은 수행하지 않았습니다.

[선택 BDS API 자료](downloads/RPG_v0560_API_server.zip?raw=true). 일반 설치는 맨 위 mcaddon 파일을 사용하면 됩니다.

이전 배포: [v0.5.59](downloads/RPG_v0559.mcaddon?raw=true).
