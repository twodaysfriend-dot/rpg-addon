# RPG v0.6.19

Bedrock 1.21.100 이상. RPG BP·Commands BP·RP 세 팩을 모두 v0.6.19로 활성화하세요.

- 흡수의 룬: 최종 흡혈 회복 비율을 절반으로 적용합니다. 일반 +0은 2.5%, +10은 6%. 강화·등급·영웅 이상 흡혈 추가 효과까지 포함해 절반입니다. 다른 기술·종족의 흡혈은 유지합니다.
- 룬 등급: 일반 → 레어 → 에픽 → 고대 → 영웅 → 전설 → 신화마다 1.5배. 힘·비전 룬은 추가 %만이 아니라 전체 피해 배율을 기준으로 증가합니다. 일반 +10의 ×1.51은 레어 ×2.265, 에픽 ×3.3975, 고대 ×5.09625가 됩니다. 영웅 이상 +8/+9/+10 추가 효과는 별도로 유지하며 전설 1.5배·신화 2.25배입니다.
- 근력·마력: 점당 피해 증가율 3.92% → 11.76%. 기본 피해 10이면 점당 +0.392 → +1.176. 고정 피해·체력·지능·직접 회복 계수는 유지합니다.
- 공용 무기 7종: 기존 평타 성장 경로를 실제 피해 처리로 검증했습니다. 등급은 기본 무기 피해에 기존 등급 배율, 강화는 단계당 피해 +1. 예: 기본 무기 피해 9라면 일반 +0 평타 10, 일반 +10 평타 20, 신화 +10 평타 29(방어·치명타 등 적용 전). 스킬에 평타 보정이 중복 적용되지 않습니다.
- 이전 v0.6.18의 모든 기능을 포함합니다. 팩 UUID를 유지하고 버전을 올려 기존 팩을 갱신합니다.

검증: JSON·JavaScript 문법·상대 import, 전체 활성 모듈 연결, 기존 integration/balance/behavior/world 검사 및 신규 룬·흡수·스탯·무기 평타 조합 검사를 실행했습니다. 실제 Minecraft 클라이언트 실행은 미검증입니다.

## 소스 검증

Python 3.12, Node.js 24.22.1. 추가 npm 패키지나 비밀키·서버 없이 실행합니다. 소스 ZIP을 별도 디렉터리에 풀고 그 위치에서 실행하세요.

```sh
python checks/validate.py .
node --experimental-vm-modules checks/integration.mjs .
node --experimental-vm-modules checks/balance.mjs .
node --experimental-vm-modules checks/behavior.mjs .
node --experimental-vm-modules checks/world.mjs .
```
