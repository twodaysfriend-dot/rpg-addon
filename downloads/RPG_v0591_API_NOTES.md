# API 확인과 확인 필요

## 공식 @minecraft/server 2.2.0 타입에서 확인

확인 자료: https://registry.npmjs.org/@minecraft/server/-/server-2.2.0.tgz 의 index.d.ts

- Player.inputPermissions, isPermissionCategoryEnabled, setPermissionCategory
- InputPermissionCategory.Movement / Camera 및 이동 하위 범주
- Entity.teleport, clearVelocity, getVelocity, applyImpulse
- EntityProjectileComponent.shoot(velocity)
- entityLoad, entitySpawn, playerDimensionChange, playerLeave, entityDie 이벤트
- world.beforeEvents.itemUse
- 2.2.0 stable에는 world.beforeEvents.entityHurt가 없음. 이에 의존하지 않음.

지원 API에서는 플레이어 이동·시점 권한을 잠그고 위치·회전을 매 틱 고정합니다.
원래 상위/하위 입력 권한 값을 보관하고 부모부터 원복합니다.
API 접근이 실패하면 둔화 255·약화 255·위치/회전 고정으로 대체합니다.
투사체는 속도를 저장하고 매 틱 고정한 뒤 projectile.shoot로 속도를 복원합니다.
몹의 효과·속도도 저장합니다. 기존 효과 지속시간은 정지 중 경과분을 빼고 복원합니다.

## 구현 범위

진짜 월드 tick 정지가 아닙니다. 레드스톤·다른 지역·전체 애니메이션은 계속됩니다.
시작 시 반경 안 적·투사체 최대 60개를 선택합니다.
정지된 몬스터가 새로 쏜 투사체는 억제하고, 골렘·외눈박이 스크립트 공격도 정지 중 차단합니다.
기존 공격 경로는 평상시 그대로이며, 정지 개체의 예약 스크립트 공격만 차단됩니다.

전용 평타·스킬은 script 피해로 처리하며 정지 대상에는 누적합니다.
화살을 들거나 시간 정지 중이면 약화 255로 중복 native 평타를 억제합니다.
시간 정지 중 다른 아이템 사용은 막습니다.
다른 애드온이 직접 가하는 피해까지 포괄하는 전역 취소 기능은 제공하지 않습니다.

## 확인 필요 — 실제 게임에서 미검증

1. Android/Windows에서 이동·시점 잠금 및 하위 입력 권한 복원.
2. 약화 255 상태에서도 entityHitEntity가 발생해 전용 평타가 발동하는지, 힘 효과/외부 애드온과 native 중복 피해가 없는지.
3. 투사체 속도·중력 복원, 정지 중 접촉 충돌. 매 틱 보정은 엔진 물리를 완전히 중단하지 못함.
4. 다른 애드온/명령어 및 정지 직전 이미 발사된 native 투사체 피해. 전용 스크립트 공격 이외의 누적은 보장하지 않음.
5. 몹의 내부 공격 타이머·모션·음향은 완전 정지가 아님.
6. 화살 1인칭/3인칭 숨김, 스탠드 UV·크기·방향. 참고 이미지 재제작 텍스처라 원본과 차이가 있음.
7. 클라이언트/플러그인 텔레포트 취소·수정 시 TPA 착지 및 지갑 복구.
8. 실제 멀티플레이의 사망·종료·차원 이동·재로드·청크 언로드 후 상태 복구.
9. 여러 플레이어가 동시에 60개체씩 정지할 때 서버 부하.

모의 테스트는 위 엔진/기기 검증을 대신하지 않습니다.

## v0.5.91 추가
시간 정지는 시전 순간 플레이어 중심 반경 20블록입니다. 정지된 플레이어의 아이템 사용을 차단하고, 남아 있는 native 피해는 피해자가 생존한 경우 되돌리는 보조 처리를 추가했습니다. stable 2.2.0에는 사전 피해 취소 이벤트가 없으므로 외부 애드온·치명적 native 특수 공격·투사체 접촉은 실제 게임 확인이 필요합니다.
