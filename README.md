# RPG 애드온 v0.5.47 — 작골염양 피해 0.7배

[설치용 애드온 다운로드](downloads/RPG_v0547.mcaddon?raw=true) · [소스와 안내](downloads/RPG_v0547_source_and_notes.zip?raw=true)

작골염양의 타격당 기본 피해를 **기존의 70%**로 조정했습니다. 검성은 **27 → 18.9**, 검사·타락검성·카멜레온은 **18 → 12.6**입니다. 3회 모두 명중하면 기본 합계는 각각 **56.7·37.8**입니다. 소수 피해를 그대로 적용하며 힘 버프·근력·장비·방어는 기존 규칙으로 계산합니다.

3회 베기·0.5초 간격·MP 50·쿨 8초·돌진·화상은 유지합니다. 검호 난도 초승달 이펙트와 월룡륜미 180도, 정보 안내, 던전 등 이전 기능도 포함합니다. 게임 내 /rpg:skillguide와 /rpg:ask 설명도 갱신했습니다.

[변경·설치 안내](docs/v0547-changes.md) · [현재 136개 스킬 도감](docs/v0547-skills.md) · [SHA-256](downloads/SHA256SUMS_v0547.txt)

행동팩·명령어팩·리소스팩을 모두 0.5.47로 갱신하세요. UUID와 저장 정보는 유지합니다.

[선택 BDS API 연결 자료](downloads/RPG_v0547_API_server.zip?raw=true) · [서버 설정 안내](docs/v0547-api.md). 기본판은 외부 API 키 없이 내장 도감으로 안내합니다.

자동 검증 291개 통과(JavaScript 286 + Python 5). JSON 446개·JS 133개와 모듈·텍스처·ZIP 무결성을 확인했습니다. 실제 Bedrock 클라이언트 전투는 미검증입니다.

이전 배포: [v0.5.46](downloads/RPG_v0546.mcaddon?raw=true).
