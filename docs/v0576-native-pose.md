# v0.5.76 바닐라 자세 보존 검증

## 바닐라 기준

39종의 교체 손 모델을 제거하고 엔진의 `minecraft:hand_equipped` 도구 렌더링을 사용합니다. 검의 기본 변환은 엔진이 담당하며 별도의 절대 각도로 흉내 내지 않습니다. `minecraft:render_offsets`는 1.20.10에서 제거된 컴포넌트이므로 사용하지 않습니다.

Mojang bedrock-samples v1.21.100.6의 `resource_pack/entity/player.entity.json`을 원본 `vanilla_player_client_base.json`에 보관했습니다. 배포 RP의 정의는 이 원본의 재질·스킨·모델·루트 및 기본 동작을 보존하고, RPG 무기를 들 때만 실행할 애니메이션 별칭 두 개와 조건 두 개를 추가합니다. 해당 자료는 Minecraft EULA 대상이며 RP의 `third_party/Mojang-bedrock-samples-LICENSE.md`에 고지를 보존했습니다.

공식 `animation.player.first_person.breathing_bob`은 네이티브 손 아이템의 위치를, `animation.player.holding_heavy_core`는 같은 손 아이템 뼈대의 배율을 조정합니다. 이번 변경도 같은 `rightItem` 뼈대에 회전 델타와 배율 2를 적용합니다. 팔·몸·스킨·기본 잡기 애니메이션은 바꾸지 않습니다.

## 추가 10°의 의미

기본 팔 회전을 G, 엔진의 기본 검/도구 변환을 T, 추가 가로 회전을 R이라고 하면, 추가 회전 D는 `G⁻¹ R G`로 계산합니다.

결과는 `G D T = R G T`이므로 원래 검 자세 `G T`가 남고 10°만 더해집니다. 이전의 `G⁻¹ × 목표 세로 자세`처럼 기본 자세를 대체하는 계산이 아닙니다. 위치 이동은 추가하지 않습니다.

1인칭의 기본 팔 각도는 공식 `player_firstperson.animation.json`의 (95, -45, 115), 3인칭 정지 잡기 각도는 `player.animation.json`의 (-18, 0, 0)을 기준으로 합니다. 이동·공격 시에는 바닐라의 원래 팔 움직임을 그대로 따릅니다. Bedrock 모델을 표시 공간으로 변환할 때 X/Y 회전 부호와 ZYX 순서를 반영합니다.

## 검사

- 기존 무기 attachable·교체 모델이 네이티브 렌더링을 덮지 않는지 확인.
- 바닐라 플레이어 정의에서 추가한 두 별칭·조건을 제외한 모든 필드가 원본과 같은지 확인.
- 주손 RPG 39종에만 적용되고 마크 기본 검·반대손·투신·격투 주먹은 제외되는지 확인.
- 임의의 기본 도구 방향 벡터에 대해 기본 자세가 유지되고 추가 회전만 10°인지 확인.
- 손 아이템 이외의 뼈대·이동값을 수정하지 않는지 확인.
- 원본 PNG·별도 투신 및 주먹 표시 보존 확인.

이는 내보낸 데이터와 좌표 계산 검사이며 실제 Bedrock 화면 캡처 검증은 아닙니다.

## 출처

- https://github.com/Mojang/bedrock-samples/tree/v1.21.100.6/resource_pack
- https://wiki.bedrock.dev/items/item-components#hand-equipped
- https://wiki.bedrock.dev/items/item-format-history#_1-20-10
