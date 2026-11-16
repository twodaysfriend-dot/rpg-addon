# RPG v0.6.47 · 미사용 데이터 정리·용의 질주 자유 조향

Bedrock 1.21.100 이상. BP·Commands BP·RP 세 팩 모두 v0.6.47로 활성화하고 월드를 다시 여세요. 팩 UUID·실제 아이템 ID·플레이어 능력치/장비는 유지합니다.

## 삭제한 항목

- 리소스팩의 scripts/ 아래 실행 등록이 없는 과거 스크립트 복사본 34개. 실제 동작하는 행동팩의 해커·스킬·샌드백 코드는 유지합니다.
- 구버전 1000×1000 마을 도안 data/fantasyCityLegacy.js.
- 이전 250×250 마을 장식 도안 data/fantasyCityV1.js.
- 구버전 마을 외곽 자동 철거/정리 core/cityTrim.js.
- 이전 도안에서 현재 도안으로 지붕/바닥을 자동 교체하던 core/cityDecor.js.
- 환수종 beast_lord 미구현 직업 항목과 전용 stub 생성 함수. 과거 강제 지정된 저장 직업은 견습으로 안전하게 읽습니다.

**파일 총 38개 삭제, 원본 비압축 크기 362,886바이트 제거**입니다. 환수종 항목/헬퍼는 남아 있는 jobs.js 안에서 삭제했으므로 38개 파일 수에 별도로 더하지 않습니다.

현재 250×250 마을 도안은 유지합니다. 새 마을 전체 생성·26개 건물·8명 전직관·지붕/바닥 장식·공간 검사·중지/재개·동일 도안의 월드 재시작 이어하기를 유지합니다.

이 패치는 기존 월드 건물·상자·아이템을 철거하거나 저장된 구버전 마을 아카이브를 삭제하지 않습니다. 구버전 자동 외곽 정리/장식 갱신 경로는 종료됩니다. 현재 도안과 다른 구버전 중단 건축은 자동 재개하지 않고 중지합니다. 필요하면 새 위치에서 /rpg:citycreate로 새 마을을 생성하세요. 현재 도안의 중단 작업은 정상 재개합니다.

## 용의 질주

매 틱 현재 시선의 수평 방향으로 이동합니다. 이동 텔레포트에서 rotation/facingLocation 강제 지정 값을 없애 카메라 입력을 덮어쓰지 않도록 수정했습니다. 중간에 90도/180도로 방향을 바꾸거나 계속 회전할 수 있습니다. 수직으로 바라보면 직전 수평 방향을 유지합니다.

초당 21칸·5초·벽 충돌/던전 경계 정지·같은 적 1회·28→42→63… 피해는 유지합니다. 질주 도중 시야각을 강제 설정하지 않는지, 중간 반전 후 이동 방향이 실제 바뀌는지 모의 API로 검사합니다. 실제 클라이언트의 카메라 입력/서버 동기화는 추가 플레이 확인이 필요합니다.

## 유지·검증 범위

- 해커의 더미데이터 스킬·훈련용 샌드백·현재 마을·몬스터/던전/보스 수치·용기사 패시브(근력 보정 기본10+현재HP10%, HP9 회복)는 유지합니다.
- 실제 스킬에 쓰이는 임시 밸런스 값은 더미로 삭제하지 않습니다. 테스트 fixture는 소스 ZIP의 검증에 사용하므로 유지하며 설치용 mcaddon에는 포함되지 않습니다.
- 새 정리 검사는 제거 경로/환수종 부재와 해커 스킬·샌드백·현재 마을 보존을 검사합니다.
- 마을 검사는 전체 생성, 재접속 이어하기, 점유 블록 보존, 구버전 자동 철거/장식 중지, 구버전 건물/아카이브 보존, 다른 도안 작업의 잘못된 재개 차단을 확인합니다.
- 기존 용기사/16개 테스트 묶음과 몬스터 1,430개 그룹·장비/종족/부활 130,560개 조합 검사를 유지합니다.

Node.js 24·Python 3.12. 내부 v0638 폴더명은 테스트/도구 경로 호환용입니다.

```sh
set -e
python3 checks/validate.py .
node checks/cleanup.mjs .
node tools/build_monster_balance.mjs --check
node checks/dungeonBalance.mjs .
python3 tools/build_abyssal_armor.py --check
node checks/largeArmor.mjs .
node --experimental-vm-modules checks/dragon.mjs .
node --experimental-vm-modules checks/city.mjs .
node --experimental-vm-modules checks/integration.mjs . --gameplay-only-wings
for suite in integration worldUnlock behavior balance world rebalance frost holyOrb vitality dragonSnow statInput wings townMuda townVillagers beastHoly michael; do
 node --experimental-vm-modules "checks/$suite.mjs" .
done
python3 tools/package.py /workspace/rpg-release/v0647
```

실제 Minecraft 엔진 실행·클라이언트 조향·렌더링·기존 월드 이행은 미검증입니다.

## 삭제 파일 전체 목록

- `RPG_BP_v0638/scripts/core/cityDecor.js`
- `RPG_BP_v0638/scripts/core/cityTrim.js`
- `RPG_BP_v0638/scripts/data/fantasyCityLegacy.js`
- `RPG_BP_v0638/scripts/data/fantasyCityV1.js`
- `RPG_RP_v0638/scripts/config.js`
- `RPG_RP_v0638/scripts/core/advance.js`
- `RPG_RP_v0638/scripts/core/commands.js`
- `RPG_RP_v0638/scripts/core/context.js`
- `RPG_RP_v0638/scripts/core/fistCoating.js`
- `RPG_RP_v0638/scripts/core/fistState.js`
- `RPG_RP_v0638/scripts/core/frenzy.js`
- `RPG_RP_v0638/scripts/core/hud.js`
- `RPG_RP_v0638/scripts/core/input.js`
- `RPG_RP_v0638/scripts/core/offer.js`
- `RPG_RP_v0638/scripts/core/playerData.js`
- `RPG_RP_v0638/scripts/core/rpgActions.js`
- `RPG_RP_v0638/scripts/core/sandbag.js`
- `RPG_RP_v0638/scripts/core/skillMenu.js`
- `RPG_RP_v0638/scripts/core/tracking.js`
- `RPG_RP_v0638/scripts/core/util.js`
- `RPG_RP_v0638/scripts/core/wings.js`
- `RPG_RP_v0638/scripts/data/jobs.js`
- `RPG_RP_v0638/scripts/data/skills.js`
- `RPG_RP_v0638/scripts/main.js`
- `RPG_RP_v0638/scripts/skills/archer.js`
- `RPG_RP_v0638/scripts/skills/berserker.js`
- `RPG_RP_v0638/scripts/skills/blood.js`
- `RPG_RP_v0638/scripts/skills/dark.js`
- `RPG_RP_v0638/scripts/skills/demon.js`
- `RPG_RP_v0638/scripts/skills/fire.js`
- `RPG_RP_v0638/scripts/skills/fist.js`
- `RPG_RP_v0638/scripts/skills/frost.js`
- `RPG_RP_v0638/scripts/skills/hacker.js`
- `RPG_RP_v0638/scripts/skills/harpoon.js`
- `RPG_RP_v0638/scripts/skills/holy.js`
- `RPG_RP_v0638/scripts/skills/mage.js`
- `RPG_RP_v0638/scripts/skills/necro.js`
- `RPG_RP_v0638/scripts/skills/sword.js`
