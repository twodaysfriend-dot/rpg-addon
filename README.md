# RPG v0.6.30 다운로드

[설치 파일 다운로드](downloads/RPG_v0630.mcaddon?raw=true) · [ZIP 다운로드](downloads/RPG_v0630_download.zip?raw=true)

Bedrock 1.21.100 이상. `.mcaddon`을 가져온 뒤 RPG BP·Commands BP·RP 세 팩을 모두 v0.6.30로 활성화하고 월드를 나갔다 다시 여세요.

- `/rpg:offer`: **일반·효과부여 책 1권당 최대 마나 +50, 다이아몬드 블록 1개당 +200.** 모든 직업에 적용하며 가방의 해당 재료 전량을 소비합니다. 기존 마나 기록은 유지하고 새 공물부터 적용합니다.
- 천사·미카엘 비행속도: `/rpg:wings 1` 기본, `/rpg:wings 2` 2배, `/rpg:wings 3` 5배.
- 숫자 인자에서 오류가 나면 **`/rpg:wings1` · `/rpg:wings2` · `/rpg:wings3`**를 사용하세요. 인자 없이 각각 기본·2배·5배로 설정합니다.
- **상태창 → 날개 비행속도**에서도 선택할 수 있습니다. 날개를 켜고 더블 점프로 비행하세요.
- 날개 명령어는 RPG 본체가 직접 등록·실행하며 Commands BP의 중계를 거치지 않습니다. `/rpg:wingcheck`와 속도 변경 성공 메시지에서 v0.6.30을 확인할 수 있습니다.
- 전직 조건 2개 완료 시 공개, 성직자·천사·미카엘의 아군 유효 회복 300 HP마다 축복의 빛 지급, 스폰·길드 기지 이동, 가방 공통 무기 강화와 이전 기능을 유지합니다.

[사용법·검증 범위](downloads/RPG_v0630_NOTES.md) · [전체 소스와 테스트](downloads/RPG_v0630_source.zip?raw=true)

12개 모의 API 테스트 묶음, 본체 단독 날개 명령어 등록 실행 및 패키지 검사 통과. 실제 Minecraft 네이티브 명령어 구문 분석·게임 실행은 미검증입니다.

SHA-256: `819139cc6deb468f657eac538999b9fb98463ff71bb1ac3ab1e3f0685466fdbb`
