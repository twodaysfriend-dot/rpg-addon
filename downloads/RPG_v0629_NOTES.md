# RPG v0.6.29 · 날개 속도 명령어·상태창 선택 수정

Bedrock 1.21.100 이상. RPG BP·Commands BP·RP 세 팩을 모두 v0.6.29로 활성화하고 월드를 다시 여세요. 기존 팩 UUID를 유지하며 v0.6.28 기능과 저장된 속도를 유지합니다.

## 수정 원인과 사용법

Bedrock Script API 2.2.0의 CustomCommand.name에는 네임스페이스가 필수입니다. v0.6.28은 `/wings`를 별도로 등록하려다 실패할 수 있었고, 모의 명령어 등록기는 이 제한을 검사하지 않았습니다. 실패하는 날개 별도 등록을 제거하고 문서와 테스트를 고쳤습니다.

천사·미카엘은 다음 명령어를 사용하세요.

- `/rpg:wings 1`: 기본 속도
- `/rpg:wings 2`: 2배 속도
- `/rpg:wings 3`: 5배 속도
- `/rpg:wings`: 기존 날개 켜기·끄기
- `/rpg:wingcheck`: 팩 버전·현재 속도·날개 장착 상태 확인

명령어 없이 **상태창 → 날개 비행속도 → 기본/2배/5배**로 설정할 수도 있습니다. 날개를 장착하고 더블 점프로 비행을 시작합니다. 속도는 개인별로 저장되고 설정만으로 날개가 켜지거나 꺼지지 않습니다. 타락 계열은 기존 속도를 유지합니다.

v0.6.28의 전직 조건 2개 달성 공개, 성직자·천사·미카엘의 아군 유효 회복 300 HP마다 축복의 빛 지급, 스폰·길드 기지 이동, 가방 공통 무기 강화는 유지합니다.

## 검증

Node.js 24·Python 3.12. 외부 패키지·비밀값·서비스는 필요 없습니다.

```sh
python3 checks/validate.py .
node tools/build_monster_balance.mjs --check
for suite in integration worldUnlock behavior balance world rebalance frost holyOrb vitality dragonSnow statInput wings; do
 node --experimental-vm-modules "checks/$suite.mjs" . || exit "$?"
done
python3 tools/package.py /tmp/rpg-release
```

모의 API 12개 테스트 묶음, JSON·JS 구문·import 경로, 1430개 몬스터 컴포넌트 그룹, 중첩 배포 ZIP 검사를 통과했습니다. 명령어 등록기가 네임스페이스 없는 이름을 거부하도록 수정했습니다. `/rpg:wings` 명령어 콜백 → 스크립트 이벤트 메시지 → 게임플레이 액션 → 저장 속도 경로와 상태창 메뉴의 1/2/5배 선택·취소를 실행했습니다. 기존 더블 점프 비행의 1/2/5배 제어 검사도 통과했습니다.

실제 Minecraft 클라이언트·서버 실행과 게임 내 메뉴·물리는 미검증입니다. `/rpg:wingcheck`에 v0.6.29가 보이지 않으면 이전 팩이 활성화된 상태입니다.
