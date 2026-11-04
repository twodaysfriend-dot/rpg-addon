# Minecraft Bedrock RPG 애드온 v0.5.33

[설치용 애드온 다운로드](downloads/RPG_v0533.mcaddon?raw=true) · [소스와 안내 ZIP](downloads/RPG_v0533_source_and_notes.zip?raw=true)

## 던전 만들기

관리자가 오버월드 빈 지역에서 실행하세요.

```mcfunction
/rpg:dungeoncreate ruin
```

위쪽 빈 공간을 검사해 43×19×43 던전을 건설합니다. 기존 블록을 덮어쓰지 않습니다. 완료 후 서바이벌에서 입장합니다.

```mcfunction
/rpg:dungeon ruin
/rpg:dungeonleave
/rpg:dungeonloot
```

최대 4인 파티, 고블린·정예 전투 방 3개와 바위 골렘 보스방을 제공합니다. 파티장이 입장시키며, 32칸 안 같은 차원의 생존 파티원이 함께 이동합니다. 클리어마다 각자 촉매 4개(매 3회차 +1), 룬 1개, 제작 재료와 추가 재화를 받습니다. 첫 클리어 모험가 갑옷 4부위. 가방 부족 보급품은 저장합니다.

## 약 20시간 풀강 목표

일반 몹 골드 114·강화석 2, 정예 360·6, 보스 3600·24로 상향했습니다. 룬 가루·기존 룬 드롭·강화 위험과 요금은 유지했습니다.

갑옷 4부위 +10과 룬 5종 +5 기준으로 각 1만 번 모의 계산했을 때 일반 사냥 평균 약 20.4시간, 던전 촉매 경로 평균 약 19.8시간입니다. 일반 사냥 5초/마리, 던전 입장·정비 포함 10분/회라는 가정이며 강화 운·실제 전투 속도에 따라 변합니다. 실제 플레이 실측이나 20시간 확정 완성 보장은 아닙니다.

- [명령어·던전·보상 상세](docs/v0533-dungeons.md)
- [풀강 비용·시간·변동 범위](docs/v0533-enhancement-costs.md)
- [직업 계열별 129개 스킬](docs/v0533-skills.md)
- [계산 원본](docs/v0533-enhancement-costs.json) · [1만 회 모의 실험](docs/v0533-progression-simulation.json)
- [파일 SHA-256](downloads/SHA256SUMS_v0533.txt)

기존 UUID를 유지하는 업데이트입니다. 월드를 복사한 뒤 설치하고 행동팩·명령어팩·리소스팩 3개가 모두 0.5.33인지 확인하세요. 파일 페이지가 열리면 Download raw file 버튼을 누르세요.

검증: JavaScript 모의 시나리오 156개와 Python 검증 5개 통과. JSON 433개·JS 117개 구문·모듈 경로·팩 의존성·ZIP 무결성 확인. 실제 Bedrock 클라이언트 실기 테스트는 수행하지 못했습니다. 일부 전역 아군 피해 차단·레벨 방어에는 기존 API 제한이 남아 있습니다.

이전 배포: [v0.5.32](downloads/RPG_v0532.mcaddon?raw=true), [v0.5.31](downloads/RPG_v0531.mcaddon?raw=true).
