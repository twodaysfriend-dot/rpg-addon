# RPG 애드온 v0.5.66 — 공용 무기·해의 도끼 표시 개선

[설치용 애드온 다운로드](downloads/RPG_v0566.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0566_source_and_notes.zip?raw=true)

공용 무기 7종과 해의 한손 도끼를 손잡이 기준으로 잡도록 조정하고, 손에 든 크기를 키웠습니다. 일반 검·적월선보다 대검·도끼를 더 크게 표시합니다. 원본 128×128 PNG는 그대로 보존합니다.

1인칭과 3인칭에 각각 자세를 적용하며, 손 움직임과 평타 모션을 따르는 결합 모델을 사용합니다. 공격력·제작법·스킬 성능은 유지합니다.

공용 무기는 모두 조합대에서 제작합니다. [7종 제작법과 변경 안내](docs/v0566-changes.md) 또는 게임 내 `/rpg:craft`를 확인하세요.

이전 팩을 비활성화하고 RPG BP·명령어 BP·RP를 모두 v0.5.66으로 갱신·활성화하세요. 기존 아이템을 다시 제작할 필요는 없습니다.

자동 검사 444개(JavaScript 439 + Python 5) 통과. JSON·스크립트 구문·import·모델 연결·원본 무기 도트·팩 무결성을 확인했습니다. 실제 모바일 Bedrock 화면에서의 자세와 크기는 미검증입니다.

[SHA-256](downloads/SHA256SUMS_v0566.txt) · [선택 BDS API 자료](downloads/RPG_v0566_API_server.zip?raw=true)

이전 배포: [v0.5.65](downloads/RPG_v0565.mcaddon?raw=true).
