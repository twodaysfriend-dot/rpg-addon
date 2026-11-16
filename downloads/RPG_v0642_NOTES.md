# RPG v0.6.42 · 필드 피해 감소 삭제·던전 피해 감소 추가 완화

Bedrock 1.21.100 이상. BP·Commands BP·RP 세 팩을 모두 v0.6.42로 활성화하고 월드를 다시 여세요. 던전은 새로 입장하세요. UUID·아이템 ID·이전 기능을 유지합니다.

## 필드 몬스터

레벨 기반 피해 감소를 전부 삭제했습니다. 22종 ×15레벨, 총 330개 필드 그룹의 받는 피해 배율을 1로 바꿨습니다. 레벨 1~260 모두 레벨 방어에 의한 피해 감소는 0%입니다. 스크립트 직접 피해/고정 피해 보정 경로도 같은 배율을 사용합니다.

이전에 소환된 필드 몬스터는 로드 시 방어 설정을 한 번 갱신합니다(밸런스 리비전 22→23). 현재 HP 비율을 보존하며 체력을 채우지 않습니다. 던전 몬스터·소환수·길들인 개체는 이 필드 갱신에서 제외합니다. 몬스터의 종별 특성(예: 히드라 화염 면역)은 유지합니다.

## 던전 몬스터

모든 단계의 일반·엘리트·보스 피해 감소율을 **v0.6.41 현재 값에서 다시 상대적으로 20% 감소**, 즉 ×0.8로 조정했습니다. v0.6.40 최초 감소율 대비 ×0.64이며, 20%p 차감이 아닙니다.

| 10단계 레벨 | v0.6.41 피해 감소 | v0.6.42 피해 감소 |
|---|---:|---:|
| 145 | 41.54% | 33.24% |
| 180 | 44.68% | 35.75% |
| 220 | 47.69% | 38.16% |
| 260 보스 | 50.23% | 40.19% |

10단계 마공학 골렘은 **체력 40,000·공격 279.14 유지**, 받는 피해 배율은 약 **0.598138**입니다. 네이티브 피해 센서와 스크립트 계산 공식을 함께 갱신했습니다.

체력·공격력·레벨·보상·제한 시간은 v0.6.41 값을 유지합니다. 심연 수호자·플레이어 방어·종족·수호 룬·스킬은 변경하지 않았습니다.

## 검증

- v0.6.41 소스와 비교해 기본 엔티티 컴포넌트 및 전체 생성 그룹의 체력/공격 값이 동일함을 확인했습니다.
- 필드 330개 그룹 피해 배율 1, 던전 1,100개 그룹 감소율 원본×0.64 및 런타임 공식 일치를 검사합니다.
- 기존 필드 리비전 22 개체가 리비전 23으로 한 번만 갱신되며 HP 비율을 보존하고 던전 개체는 제외되는지 검사합니다.
- 1~10단계 보스 체력 증가 및 최종 체력 40,000 검사를 유지합니다.

Node.js 24·Python 3.12. 내부 v0638 폴더명은 테스트/도구 경로 호환용입니다.

```sh
set -e
python3 checks/validate.py .
node tools/build_monster_balance.mjs --check
node checks/dungeonBalance.mjs .
python3 tools/build_abyssal_armor.py --check
node checks/largeArmor.mjs .
node --experimental-vm-modules checks/integration.mjs . --gameplay-only-wings
for suite in integration worldUnlock behavior balance world rebalance frost holyOrb vitality dragonSnow statInput wings townMuda townVillagers beastHoly michael; do
 node --experimental-vm-modules "checks/$suite.mjs" .
done
python3 tools/package.py /workspace/rpg-release/v0642
```

수치/모의 API 검증과 실제 Minecraft 엔진 검증은 구분합니다. 실제 게임 실행·체감 난이도·기존 월드 이행은 추가 확인이 필요합니다.
