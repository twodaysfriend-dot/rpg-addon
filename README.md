# RPG 애드온 v0.5.73 — 무기별 손잡이·착용 자세 교정

[설치용 애드온 다운로드](downloads/RPG_v0573.mcaddon?raw=true) · [소스와 검증 자료](downloads/RPG_v0573_source_and_notes.zip?raw=true)

무기가 손 아래에 뜨거나 머리 쪽으로 벗어나던 좌표 오류를 수정했습니다. 보이는 공용·직업 아이템 39종의 손잡이·방향점을 원본 그림에서 직접 지정했습니다. 검·도끼·창·낫·지팡이는 자루, 활은 중앙 그립, 법서·코어는 하단을 기준으로 잡습니다.

손 바인딩 높이 보정과 앞뒷면 UV의 좌우 반전을 함께 계산합니다. 삼지창의 이동값과 실루엣 자동 추정을 제거하고, 손잡이를 중심으로 회전하도록 구성했습니다. 1인칭 팔 회전도 Bedrock 축·회전 순서로 보정했습니다.

**원본 도트와 2배 모델 크기는 유지합니다.** 투신의 본인 전용 표시와 보이지 않는 격투 주먹도 유지합니다. 전투·결투·PK·강화·스킬 수치는 바꾸지 않았습니다.

[이번 변경 사항](docs/v0573-changes.md) · [원인·좌표 검증 방식](docs/v0573-hand-validation.md) · [SHA-256](downloads/SHA256SUMS_v0573.txt)

자동 검사 500개(JavaScript 495 + Python 5)를 통과했습니다. 내보낸 모델의 실제 UV를 읽어 39종 앞뒷면의 손잡이가 손 위치에 놓이는지, 카메라·팔 회전 후에도 따라오는지 검사합니다. 실제 Bedrock 모바일·Realm 화면은 실행 검증하지 못했습니다. 소스 자료의 `review/` 그림은 좌표 투영 도식이며 Minecraft 스크린샷이 아닙니다.

**기존 v0.5.72 팩을 비활성화하고 RPG BP·RPG Commands BP·RPG RP 모두 v0.5.73으로 적용한 뒤 월드를 다시 여세요.** 팩 UUID와 기존 성장·장비·카멜레온 기술·지갑·PK 저장 키는 유지합니다.

[선택 BDS API 자료](downloads/RPG_v0573_API_server.zip?raw=true) · [이전 v0.5.72](downloads/RPG_v0572.mcaddon?raw=true)
