# RPG 애드온 v0.5.76 — 기본 검 자세 + 가로 10°

[설치용 애드온 다운로드](downloads/RPG_v0576.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0576_source_and_notes.zip?raw=true)

**마크 기본 도구/검의 실제 잡는 자세를 유지하고, 거기서 가로로 10°만 더 기울입니다.** 기존처럼 화면의 세로축에서 10°가 되는 자세를 새로 만들지 않습니다.

공용·직업 표시 아이템 39종의 교체 손 모델을 제거하고 엔진의 `minecraft:hand_equipped` 렌더링을 사용합니다. 바닐라의 잡기·휘두르기·걷기·아이템 교체 동작을 유지한 채 손 아이템 뼈대에만 추가 회전과 2배 배율을 적용합니다.

원본 도트·아이콘·아이템 ID·공격력·강화·등급·제작·직업·스킬·PvP·PK·지갑·성장 데이터는 유지합니다. 일반 마크 아이템, 투신 전용 표시와 보이지 않는 격투 주먹은 이번 추가 회전·배율 대상에서 제외합니다.

[변경 사항](docs/v0576-changes.md) · [바닐라 기준·회전 검증](docs/v0576-native-pose.md) · [SHA-256](downloads/SHA256SUMS_v0576.txt)

자동 검사 501개(JavaScript 496 + Python 5)를 통과했습니다. 실제 Bedrock 모바일·Realm 화면은 직접 실행 검증하지 못했습니다.

**이전 RP를 모두 끄고 BP·명령어 BP·RP 세 팩을 v0.5.76으로 적용한 뒤 월드를 다시 여세요.** 이전 RP가 활성화돼 있으면 그 팩의 별도 무기 모델이 다시 적용될 수 있습니다. 팩 UUID와 기존 저장 데이터 식별자는 유지했습니다.

[선택 BDS API 자료](downloads/RPG_v0576_API_server.zip?raw=true) · [이전 v0.5.75](downloads/RPG_v0575.mcaddon?raw=true)
