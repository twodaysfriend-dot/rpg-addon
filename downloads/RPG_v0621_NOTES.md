# RPG v0.6.21

Bedrock 1.21.100 이상. RPG BP·Commands BP·RP 세 팩을 모두 v0.6.21로 적용하세요.

## 스탯 입력

스탯 배분 시 나타나던 “1 이상의 정수를 입력하세요” 경고 문구를 제거했습니다. 빈칸·0·음수·소수·문자 등 유효하지 않은 입력은 포인트를 소비하지 않고 조용히 스탯 화면으로 돌아갑니다. 정상적인 양의 정수는 기존대로 배분됩니다. 전각 숫자(예: １２)도 일반 숫자로 인식합니다. 잔여 SP보다 많은 배분은 여전히 거부합니다.

v0.6.20의 집 26곳 건축 허용·마을 금블록/성/성벽 보호·황금 주먹 무다무다와 이전 룬·스탯 밸런스 기능을 모두 포함합니다.

## 검증

JSON·JavaScript 문법/상대 import 검사, 전체 활성 모듈 연결을 포함한 integration 검사와 실제 스탯 화면 흐름의 statInput 검사를 통과했습니다. 일반/전각/공백 포함/숫자 입력 배분, 경고 제거, 잘못된 입력·취소 시 SP 보존, 잔여 SP 초과 거부, 마력 배분을 확인했습니다. 실제 Minecraft 실행은 미검증입니다.

소스 ZIP을 풀고 Python 3.12와 Node.js 24에서 실행하세요. 추가 npm 패키지는 필요 없습니다.

```sh
python3 checks/validate.py .
node --experimental-vm-modules checks/integration.mjs .
node --experimental-vm-modules checks/statInput.mjs .
```
