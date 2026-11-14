# RPG v0.6.29 다운로드

[설치 파일 다운로드](downloads/RPG_v0629.mcaddon?raw=true) · [ZIP 다운로드](downloads/RPG_v0629_download.zip?raw=true)

Bedrock 1.21.100 이상. `.mcaddon`을 가져온 뒤 RPG BP·Commands BP·RP 세 팩을 모두 v0.6.29로 활성화하고 월드를 다시 여세요.

천사·미카엘 비행속도는 **`/rpg:wings 1` 기본 · `/rpg:wings 2` 2배 · `/rpg:wings 3` 5배**입니다. 상태창의 **날개 비행속도**에서도 선택할 수 있습니다. 날개를 켜고 더블 점프로 비행하세요.

v0.6.28에서 `/wings` 등록 실패를 허용하던 코드를 수정했습니다. Bedrock Script API는 명령어 이름에 네임스페이스를 요구하므로 `/rpg:wings`를 사용하세요. `/rpg:wingcheck`로 v0.6.29와 현재 속도를 확인할 수 있습니다.

전직 조건 2개 완료 시 조건 공개, 성직자·천사·미카엘의 아군 유효 회복 300 HP마다 축복의 빛 지급, 스폰·길드 기지 이동, 가방 공통 무기 강화 및 모든 이전 기능을 유지합니다.

[사용법·검증 범위](downloads/RPG_v0629_NOTES.md) · [전체 소스와 테스트](downloads/RPG_v0629_source.zip?raw=true)

네임스페이스를 검사하는 명령어 등록·실제 콜백 전달 경로·상태창 속도 선택을 포함한 모의 API 12개 테스트 묶음 및 패키지 검사를 통과했습니다. 실제 Minecraft 실행은 미검증입니다.

SHA-256: `8f56962b05c1a071e0aa0e19cd28cb21c89af51118147076f864f42f7b354572`
