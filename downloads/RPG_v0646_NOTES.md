# RPG v0.6.46 · 용기사 3타 패시브 변경

Bedrock 1.21.100 이상. BP·Commands BP·RP 모두 v0.6.46로 활성화하고 월드를 다시 여세요. UUID·아이템 ID·직업/스킬 목록을 유지합니다.

## 변경된 패시브

첫 두 타에는 패시브 추가 피해/회복이 없고 **세 번째 유효 타격**에만 다음을 적용합니다.

**추가 피해 = 기본 물리 10에 근력·힘 효과·물리 룬·직업 무기 강화를 보정한 값 + 시전자의 패시브 회복 전 현재 HP ×10%**

피해 적용 뒤 **자신의 HP 9(하트 4.5칸)를 회복**합니다. 최대 HP를 넘겨 회복하지 않습니다. 체력 비례 계수는 별도 수치 지정이 없어 10%를 기본값으로 선택했습니다. 최대 체력이 아닌 발동 시점의 현재 체력 기준이며 HP 9 회복 이전에 한 번만 계산합니다. HP 비례분에 근력/룬/무기 배율을 다시 곱하지 않습니다.

예: 물리 보정 배율 3배, 현재 HP 70이라면 10×3+70×0.1=37 추가 피해 후 HP 70→79. 현재 HP 98이라면 39.8 추가 피해 후 HP 98→100(최대100 기준)입니다.

## 발동·기존 기능

- 카운터는 용기사 플레이어별로 누적되며, 서로 다른 적을 두 번 때리고 세 번째 적을 때리면 세 번째 적에게 발동합니다. 발동 뒤 카운터는 0으로 돌아갑니다.
- 피해가 실제로 양수인 entityAttack 이벤트만 집계합니다. 피해 0·override 원인·사망한 시전자는 집계하지 않습니다. 기존 코드처럼 일부 entityAttack 기반 스킬도 포함될 수 있습니다. 질주는 override이므로 제외됩니다.
- 카멜레온이 삼조격을 배워도 용기사 고유 패시브는 얻지 않습니다.
- 액티브 삼조격의 추가 피해 24×3회·3타 기절은 유지합니다. 패시브와 함께 세 번째 타격에 각각 적용할 수 있습니다.
- 패시브 피해는 기존 직접 HP 감소 경로를 유지합니다. 공통 전투 처리의 종족·심연 갑옷·부활·마을 주민 보호 규칙도 거칩니다.
- 용의 시선40·무쌍돌격36·방향 조정 질주28부터·브레스 마법72·현월수룡50+매초10/나약함II/외부 엔티티 투사체 삭제 등 v0.6.45 기능을 유지합니다.
- 던전/보스/필드 몬스터 수치와 플레이어의 다른 능력은 변경하지 않았습니다.

설정: scripts/data/skills.js의 DRAGON_PASSIVE `{baseDamage:10,currentHealthRatio:0.1,heal:9}`. 발동 처리: skills/dragon.js. 게임 내 직업/스킬 안내도 갱신했습니다.

## 검증

새 패시브 회귀 검사에서 첫 2타 미발동·3타 발동·근력 보정 1회·현재 HP 비례분의 추가 근력 보정 방지·회복 전 HP 사용·HP 9 회복·최대 체력 제한·낮은/높은 현재 HP·카운터 초기화·다른 적으로 누적·피해0/override/사망/카멜레온 제외를 확인합니다. 삼조격 24×3과 패시브가 겹치는 경로도 검사합니다.

Node.js 24·Python 3.12. 내부 v0638 폴더명은 기존 도구·테스트 경로 호환용입니다.

```sh
set -e
python3 checks/validate.py .
node tools/build_monster_balance.mjs --check
node checks/dungeonBalance.mjs .
python3 tools/build_abyssal_armor.py --check
node checks/largeArmor.mjs .
node --experimental-vm-modules checks/dragon.mjs .
node --experimental-vm-modules checks/integration.mjs . --gameplay-only-wings
for suite in integration worldUnlock behavior balance world rebalance frost holyOrb vitality dragonSnow statInput wings townMuda townVillagers beastHoly michael; do
 node --experimental-vm-modules "checks/$suite.mjs" .
done
python3 tools/package.py /workspace/rpg-release/v0646
```

실제 Minecraft 엔진 실행·전투 체감·렌더링·기존 월드 이행은 미검증입니다.
