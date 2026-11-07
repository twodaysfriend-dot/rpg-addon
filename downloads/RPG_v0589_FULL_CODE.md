# 수정·신규 파일 전체 코드

v0.5.88 대비 변경된 텍스트 파일을 생략 없이 수록합니다. PNG는 ZIP의 같은 경로에 포함됩니다.

## RPG_BP_v0578/entities/the_world_stand.json

```json
{
  "format_version": "1.21.0",
  "minecraft:entity": {
    "description": {
      "identifier": "rpg:the_world_stand",
      "is_spawnable": false,
      "is_summonable": true
    },
    "components": {
      "minecraft:type_family": {
        "family": [
          "rpg_cosmetic"
        ]
      },
      "minecraft:collision_box": {
        "width": 0,
        "height": 0
      },
      "minecraft:physics": {
        "has_gravity": false,
        "has_collision": false
      },
      "minecraft:pushable": {
        "is_pushable": false,
        "is_pushable_by_piston": false
      },
      "minecraft:damage_sensor": {
        "triggers": [
          {
            "cause": "all",
            "deals_damage": false
          }
        ]
      },
      "minecraft:persistent": {}
    }
  }
}
```

## RPG_BP_v0578/items/the_world_arrow.json

```json
{
  "format_version": "1.21.90",
  "minecraft:item": {
    "description": {
      "identifier": "rpg:the_world_arrow",
      "menu_category": {
        "category": "items"
      }
    },
    "components": {
      "minecraft:max_stack_size": 1,
      "minecraft:hand_equipped": true,
      "minecraft:icon": "rpg_the_world_arrow",
      "minecraft:damage": 0,
      "minecraft:display_name": {
        "value": "더 월드의 화살 · 전용 건틀릿"
      },
      "rpg:cast_item": {}
    }
  }
}
```

## RPG_BP_v0578/manifest.json

```json
{
  "format_version": 2,
  "header": {
    "name": "RPG Addon BP v0.5.89",
    "description": "RPG v0.5.89 · 더 월드·시간 정지·스탠드·1000골드 TPA",
    "uuid": "a1b49558-2a78-4754-bae7-1c5ec7a84303",
    "version": [
      0,
      5,
      89
    ],
    "min_engine_version": [
      1,
      21,
      100
    ]
  },
  "modules": [
    {
      "type": "data",
      "uuid": "cc5d6e3c-5285-4787-8a59-a8e2856f2aec",
      "version": [
        0,
        5,
        89
      ]
    },
    {
      "type": "script",
      "language": "javascript",
      "uuid": "c41fa3d6-d82c-4a54-9783-68f92fe633a1",
      "entry": "scripts/gameplayEntry.js",
      "version": [
        0,
        5,
        89
      ]
    }
  ],
  "dependencies": [
    {
      "uuid": "8fa45523-e936-4258-a195-01e5e62e814c",
      "version": [
        0,
        5,
        89
      ]
    },
    {
      "module_name": "@minecraft/server",
      "version": "2.2.0"
    },
    {
      "module_name": "@minecraft/server-ui",
      "version": "2.0.0"
    }
  ]
}
```

## RPG_BP_v0578/scripts/core/advance.js

```javascript
import { jobTrialStatus } from './jobTrials.js';
import {upgradeLevel} from '../data/upgrades.js';
import {setGearLevel} from './gearState.js';
import { ItemStack, ItemLockMode } from "@minecraft/server";
import { ActionFormData,MessageFormData } from "@minecraft/server-ui";
import { JOB_TRIALS } from '../data/jobTrials.js';
import { JOBS, ALL_WEAPON_IDS, STARTER_BY_LINE, APPRENTICE_SKILLS } from "../data/jobs.js";
import { SKILLS } from "../data/skills.js";
import {
  getJobId, setJobId, getSkills, setSkills, setMana, setIndex, maxManaOf, getProgress, getDisabled, setDisabled,addProgress,
} from "./playerData.js";
import { sound } from "./util.js";
import { showForm } from "./skillMenu.js";
import { beginnerAdvanceUnlocked } from './beginnerData.js';
import { ALL_EQUIPMENT_IDS } from '../data/equipment.js';
import { recordBasicJobAdvance } from './newTrials.js';

// 전직 조건 진행도 (조건이 없으면 null)
export function requirementStatus(player, target) {
  if(target?.unlockPending)return {met:false,have:0,need:1,label:'해금 조건 미정 · 관리자 /rpg:force the_world로 시험 가능'};
  if(target?.id==='hacker')return {met:false,have:0,need:1,label:'로토와 23회 대화 후 퀴즈 정답 · 월드 최초 정답자 1명만 전직'};
  const trial=jobTrialStatus(player,target);if(trial)return trial;
  if (!target?.req) return null;
  if (beginnerAdvanceUnlocked(player, target)) return {have:5, need:5, label:'첫 사냥 (모험 안내)', met:true};
  const have = Math.min(getProgress(player)[target.req.key] ?? 0, target.req.need);
  return { have, need: target.req.need, label: target.req.label, met: have >= target.req.need };
}

// 전직 가능 여부: 직전 직업 + 전직 조건
export function checkAdvance(player, target) {
  if(target?.unlockPending)return {ok:false,reason:'더 월드는 해금 조건이 미정입니다. 현재 관리자 강제 전직만 지원합니다.'};
  if (!target) return { ok: false, reason: "존재하지 않는 직업입니다." };
  if(target.id==='hacker')return {ok:false,reason:'해커는 로토와 23회 대화 후 퀴즈를 맞힌 월드 최초 1명만 전직할 수 있습니다. 로토에게 말을 걸어 주세요.'};

  if (target.placeholder) {
    return { ok: false, reason: `${target.name}은(는) 아직 구현되지 않았습니다.` };
  }
  const cur = getJobId(player);
  const hiddenStart=target.hidden&&target.parent===null;
  if(hiddenStart&&((JOBS[cur]?.tier??0)<2)&&!(cur==='apprentice'&&JOB_TRIALS[target.id]?.fromApprentice))return {ok:false,reason:'히든 직업은 2차 이상 직업에서 시작할 수 있습니다.'};
  const allowedFrom=JOB_TRIALS[target.id]?.allowedFrom;
  const renouncedHoly=target.id==='fallen_angel'&&cur==='apprentice'&&player.getDynamicProperty('rpg:renounced_holy')===true;
  if (allowedFrom ? !allowedFrom.includes(cur)&&!renouncedHoly : !hiddenStart && target.parent !== cur) {
    const from = allowedFrom?.map(id=>JOBS[id].name).join("·") ?? JOBS[target.parent]?.name ?? "?";
    return { ok: false, reason: `${target.name}은(는) ${from}에서만 전직할 수 있습니다.` };
  }

  const st = requirementStatus(player, target);
  if (st && !st.met) {
    return {
      ok: false,
      reason: `전직 조건 미달: ${st.label} ${st.have}/${st.need}`,
    };
  }
  return { ok: true };
}

// 2차 전직 때 사라질 스킬 id 목록 (선택한 계열 밖의 스킬)
export function skillsLostOnAdvance(player, target) {
  if (target.tier !== 2) return [];
  return getSkills(player).filter((id) => SKILLS[id]?.line !== target.line);
}

function ensureMoonFocus(player, container) {
  if (getJobId(player) !== "moon_heir") return;
  for (let i=0;i<container.size;i++) if (container.getItem(i)?.typeId === "rpg:moon_heir_focus") return;
  const focus = new ItemStack("rpg:moon_heir_focus", 1);
  focus.keepOnDeath = true;
  const remainder = container.addItem(focus);
  if (remainder) player.dimension.spawnItem(remainder, player.location);
}

function moveOrDrop(player, container, item) {
  for (let i = 1; i < container.size; i++) {
    if (!container.getItem(i)) {
      container.setItem(i, item);
      return;
    }
  }
  player.dimension.spawnItem(item, player.location);
}

// 직업 무기를 1번 칸에 고정해서 지급합니다. 이전 RPG 무기(견습 시작 아이템 포함)는 모두 회수합니다.
export function giveWeapon(player, job) {
  const c = player.getComponent("minecraft:inventory")?.container;
  if (!c) return;
  const previous=c.getItem(0),matching=Array.from({length:c.size},(_,i)=>c.getItem(i)).find(i=>i?.typeId===job.weapon);
  const preserved=matching?.clone(),inherited=Math.max(upgradeLevel(matching,'weapon'),upgradeLevel(ALL_WEAPON_IDS.includes(previous?.typeId)?previous:undefined,'weapon'));

  for (let i = 0; i < c.size; i++) {
    const it = c.getItem(i);
    if (it && (ALL_WEAPON_IDS.includes(it.typeId) || it.typeId === "rpg:moon_heir_focus")) c.setItem(i, undefined);
  }
  if (!job.weapon) return;

  const old = c.getItem(0);
  if (old) {
    c.setItem(0, undefined);
    moveOrDrop(player, c, old);
  }

  const weapon = preserved&&upgradeLevel(preserved,'weapon')>=inherited?preserved:setGearLevel(preserved??new ItemStack(job.weapon, 1),'weapon',inherited);
  weapon.lockMode = ItemLockMode?.slot ?? "slot";
  weapon.keepOnDeath = true;
  c.setItem(0, weapon);
  if (job.id === "moon_heir") ensureMoonFocus(player, c);
}

// 1번 칸에 무기가 없으면 다시 지급 (사망/크리에이티브 조작 등 대비)
export function ensureWeapon(player) {
  const job = JOBS[getJobId(player)];
  if (!job?.weapon) return;
  const c = player.getComponent("minecraft:inventory")?.container;
  if (!c) return;
  if (c.getItem(0)?.typeId === job.weapon || (job.id === "chameleon" && c.getItem(0)?.typeId === job.altWeapon)) {
    if (job.id === "moon_heir") ensureMoonFocus(player, c);
    return;
  }
  giveWeapon(player, job);
}

// 견습 시작 아이템 지급 + 그 계열의 견습 스킬 습득 (초반 지역의 상자/NPC/명령 블록에서 호출)
export function giveStarter(player, line) {
  const itemId = STARTER_BY_LINE[line];
  if (!itemId) return { ok: false, reason: "알 수 없는 계열입니다. (sword, mage, holy, fist, archer)" };
  if (getJobId(player) !== "apprentice") {
    return { ok: false, reason: "견습만 받을 수 있습니다." };
  }

  const learned = getSkills(player);
  for (const id of APPRENTICE_SKILLS[line] ?? []) {
    if (!learned.includes(id)) learned.push(id);
  }
  setSkills(player, learned);

  const c = player.getComponent("minecraft:inventory")?.container;
  if (c) {
    let has = false;
    for (let i = 0; i < c.size; i++) if (c.getItem(i)?.typeId === itemId) has = true;
    if (!has) {
      const left = c.addItem(new ItemStack(itemId, 1));
      if (left) player.dimension.spawnItem(left, player.location);
    }
  }
  if (line === "archer" && c) {
    const arrows = c.addItem(new ItemStack("minecraft:arrow", 64));
    if (arrows) player.dimension.spawnItem(arrows, player.location);
  }
  return { ok: true };
}

// 애드온 전용 아이템 전부를 인벤토리에 지급 (이미 가진 아이템은 건너뜀)
export function giveAllItems(player) {
  const c = player.getComponent("minecraft:inventory")?.container;
  if (!c) return 0;
  const have = new Set();
  for (let i = 0; i < c.size; i++) {
    const it = c.getItem(i);
    if (it) have.add(it.typeId);
  }
  let n = 0;
  for (const id of new Set([...ALL_WEAPON_IDS, ...ALL_EQUIPMENT_IDS, "rpg:blessing_light", "rpg:holy_water", "rpg:sandbag_spawner", "rpg:angel_wing_toggle", "rpg:michael_wing_toggle", "rpg:fallen_wing_toggle", "rpg:mangnani_contract", "rpg:blood_spear"])) {
    if (have.has(id)) continue;
    const left = c.addItem(new ItemStack(id, 1));
    if (left) player.dimension.spawnItem(left, player.location);
    n++;
  }
  return n;
}

export function applyAdvance(player, target) {
  let skills = getSkills(player);
  if(target.id==='fallen_angel'){skills=[];setDisabled(player,[]);}
  if (target.tier === 2) {
    skills = skills.filter((id) => SKILLS[id]?.line === target.line);
  }
  if (target.tier >= 3) {
    // 견습 마법처럼 낮은 차수까지만 쓰는 스킬은 목록에서 제외
    skills = skills.filter((id) => (SKILLS[id]?.maxTier ?? 99) >= target.tier);
  }
  // 천궁 전직 시 궁수 전용 불 화살이 폭탄 화살로 교체됩니다.
  if (target.id === "heavenly_archer") {
    skills = skills.filter(id => id !== "archer_fire" && id !== "archer_triple");
    const disabled = getDisabled(player);
    if (disabled.includes("archer_fire")) setDisabled(player, disabled.filter(id => id !== "archer_fire"));
  }
  // 검성 계열 전직 시 발도를 제거합니다. 섬멸은 검성만 보유합니다.
  if (target.id === "sword_saint" || target.id === "fallen_sword_saint") {
    skills = skills.filter((id) => id !== "sword_draw");
    const off = getDisabled(player);
    if (off.includes("sword_draw")) setDisabled(player, off.filter((id) => id !== "sword_draw"));
  }
  if (target.id === "fallen_sword_saint") {
    skills = skills.filter((id) => id !== "sword_annihilation");
    const off = getDisabled(player);
    if (off.includes("sword_annihilation")) setDisabled(player, off.filter((id) => id !== "sword_annihilation"));
  }
  for (const id of target.skills) if (!skills.includes(id)) skills.push(id);

  setSkills(player, skills);
  setJobId(player, target.id);
  try{recordBasicJobAdvance(player,target.id);}catch{}
  if(target.id==='fallen_angel')player.setDynamicProperty('rpg:renounced_holy',undefined);
  setIndex(player, 0);
  setMana(player, maxManaOf(player));
  giveWeapon(player, target);
  if (target.line === "archer") {
    try {
      const c = player.getComponent("minecraft:inventory")?.container;
      const arrows = c?.addItem(new ItemStack("minecraft:arrow", 64));
      if (arrows) player.dimension.spawnItem(arrows, player.location);
    } catch { /* 인벤토리가 준비되지 않은 경우 */ }
  }

  try {
    player.onScreenDisplay.setTitle(`${target.corrupt ? "§c" : "§b"}${target.name}`, {
      subtitle: "전직 완료",
      fadeInDuration: 10,
      stayDuration: 50,
      fadeOutDuration: 10,
    });
  } catch {
    player.sendMessage(`§e전직 완료: ${target.name}`);
  }
  sound(player.dimension, "random.levelup", player.location, { volume: 1, pitch: 1 });
}

export function resetToApprentice(player) {
  setJobId(player, "apprentice");
  setSkills(player, []);
  setDisabled(player, []);
  setIndex(player, 0);
  setMana(player, maxManaOf(player));
  giveWeapon(player, JOBS.apprentice);
}

// 전직 요청. 사라지는 스킬이 있으면 확인창을 먼저 띄웁니다.
// force=true 는 테스트용으로 조건 검사를 건너뜁니다.
export async function requestAdvance(player, target, force = false) {
  if(!force&&target?.id==='cleric'&&getJobId(player)==='apprentice'){
    if((Number(player.getDynamicProperty('rpg:rpg_level'))||1)<10){player.sendMessage('§e성직자 전직은 레벨 10부터입니다.');return;}
    const answer=await showForm(new MessageFormData().title('성직 전직관').body('회복과 지원의 길을 선택하시겠습니까?\n성직자로 전직하며 다른 계열의 견습 기술은 정리됩니다.').button1('아직 결정하지 않음').button2('확실하게 성직자로 전직'),player);
    if(!answer||answer.canceled||answer.selection!==1)return;
    if(getJobId(player)!=='apprentice')return;
    addProgress(player,'holy_consulted',1);
  }
  if (!force) {
    const r = checkAdvance(player, target);
    if (!r.ok) {
      player.sendMessage(`§c${r.reason}`);
      return;
    }
  }
  if(!force&&target?.id==='fallen_angel'){
    const answer=await showForm(new MessageFormData().title('성직 포기 · 타락천사').body('치유한 플레이어의 사망을 다섯 번 목격했습니다.\n성직을 포기하고 기존 성직 기술을 잃은 뒤 타락천사로 전직합니다.').button1('성직 유지').button2('성직을 포기하고 타락'),player);
    if(!answer||answer.canceled||answer.selection!==1)return;
    const current=checkAdvance(player,target);if(!current.ok){player.sendMessage(current.reason);return;}
  }

  const lost = skillsLostOnAdvance(player, target);
  if (lost.length === 0) {
    applyAdvance(player, target);
    return;
  }

  const names = lost.map((id) => SKILLS[id]?.name ?? id).join(", ");
  const form = new ActionFormData()
    .title("전직 확인")
    .body(
      `${target.name}(으)로 전직하면 다음 스킬이 사라집니다.\n\n§c${names}§r\n\n되돌릴 수 없습니다. 전직할까요?`
    )
    .button("전직한다")
    .button("취소");

  const res = await showForm(form, player);
  if (!res || res.canceled) {
    player.sendMessage("§7전직창이 닫혔습니다. 채팅창을 닫고 다시 시도해 주세요.");
    return;
  }
  if (res.selection !== 0) return;

  // 확인창이 떠 있는 동안 상태가 바뀌었을 수 있으니 다시 확인
  if (!force) {
    const r = checkAdvance(player, target);
    if (!r.ok) {
      player.sendMessage(`§c${r.reason}`);
      return;
    }
  }
  applyAdvance(player, target);
}
```

## RPG_BP_v0578/scripts/core/combatContext.js

```javascript
import {isTimeFrozen} from './timeStopDamage.js';
import {system} from '@minecraft/server';
import {applyDuelDamage,pvpBlocked,notePvpAttack} from './pvp.js';
export function setCombatHealth(target,source,hp,next){if(isTimeFrozen(source))return;if(pvpBlocked(source,target))return;const amount=Math.max(0,hp.currentValue-next),handled=applyDuelDamage(source,target,amount);if(!handled){notePvpAttack(source,target,amount);if(next<=0&&source)rememberScriptKiller(source,target);hp.setCurrentValue(next);}}
export const BLACK_FLASH_KEY='rpg:black_flash_ready';
const hits=[],pending=[],credits=new Map();let onManual=()=>{},basicHandler=()=>{},manualObserver=()=>{},manualMultiplier=()=>1;
export function setManualDamageMultiplier(fn){manualMultiplier=fn;}
export function setManualDamageObserver(fn){manualObserver=fn;}
export function setUnreportedDamageHandler(fn){onManual=fn;}
export function setBasicHitHandler(fn){basicHandler=fn;}
export function handleBasicHit(ev,scripted){basicHandler(ev,scripted);}
const same=(h,s,t)=>h.source?.id===s?.id&&h.target?.id===t?.id;
export function currentScriptHit(source,target){return [...hits].reverse().find(h=>same(h,source,target));}
function prune(){for(let i=pending.length-1;i>=0;i--)if(pending[i].until<system.currentTick)pending.splice(i,1);}
export function markDamageReported(source,target){
 prune();const h=currentScriptHit(source,target)??pending.find(h=>same(h,source,target));if(h){h.reported=true;const i=pending.indexOf(h);if(i>=0)pending.splice(i,1);}return h;
}
export function noteManualDamage(source,target,damage,{incomingScaled=false}={}){
 if(pvpBlocked(source,target))return;
 if(!incomingScaled&&damage>0){try{const hp=target.getComponent('minecraft:health'),extra=Math.min(hp?.currentValue??0,damage*Math.max(0,manualMultiplier(target)-1));if(extra>0){if(extra>=hp.currentValue&&source)rememberScriptKiller(source,target);setCombatHealth(target,source,hp,Math.max(0,hp.currentValue-extra));damage+=extra;}}catch{}}
 const h=currentScriptHit(source,target);if(h)h.manualDamage=(h.manualDamage??0)+damage;try{manualObserver(source,target,damage);}catch{}
}

export function rememberScriptKiller(source,target){for(const [id,c]of credits)if(c.until<system.currentTick)credits.delete(id);credits.set(target.id,{source,until:system.currentTick+4});}
export function scriptKiller(target){const c=credits.get(target?.id);credits.delete(target?.id);return c&&c.until>=system.currentTick?c.source:undefined;}
export function scriptedHit(target,source,fn){
 if(isTimeFrozen(source))return false;
 let before;try{before=target.getComponent('minecraft:health')?.currentValue;}catch{}
 prune();const h={source,target,before,reported:false,until:system.currentTick+2};hits.push(h);pending.push(h);let result;
 try{result=fn();return result;}finally{
  hits.pop();if(h.manualDamage>0){try{onManual(source,target,h.manualDamage);}catch{}h.reported=true;}
  if(h.reported||result===false){const i=pending.indexOf(h);if(i>=0)pending.splice(i,1);}
 }
}
```

## RPG_BP_v0578/scripts/core/cyclopsBoss.js

```javascript
import {isTimeFrozen} from './timeStopDamage.js';
import {levelAttackMultiplier,MONSTER_LEVEL_KEY} from '../data/monsterLevels.js';
import {playBossMotion} from './bossMotion.js';
import { world, system, EntityDamageCause } from '@minecraft/server';

export const CYCLOPS_ID='rpg:cyclops_boss';
export const EYE_HEIGHT=5.44;
export const EYE_FORWARD=1.16;
export const EYE_RADIUS=0.67;
const STATE=new Map();
const PROJECTILE_EYES=new Map();
const BONUS_IN_FLIGHT=new Set();
const DUST='minecraft:basic_smoke_particle';
const CRIT='minecraft:critical_hit_emitter';
function valid(e){try{return !!e?.isValid && (e.getComponent('minecraft:health')?.currentValue??0)>0;}catch{return false;}}
function vec(a,b){return {x:a.x-b.x,y:a.y-b.y,z:a.z-b.z};}
function dist(a,b){const d=vec(a,b);return Math.hypot(d.x,d.y,d.z);}
function flatView(boss){
  try{const v=boss.getViewDirection();const n=Math.hypot(v.x,v.z)||1;return {x:v.x/n,z:v.z/n};}
  catch{return {x:0,z:-1};}
}
export function eyeCenter(boss){
  const l=boss.location,v=flatView(boss);
  return {x:l.x+v.x*EYE_FORWARD,y:l.y+EYE_HEIGHT,z:l.z+v.z*EYE_FORWARD};
}
export function isEyeProjectileHit(boss,location,source){
  if(!location || source?.typeId!=='minecraft:player')return false;
  const eye=eyeCenter(boss),f=flatView(boss),back=vec(source.location,eye);
  return (back.x*f.x+back.z*f.z)>-0.1 && dist(location,eye)<=0.92;
}
export function isEyeMeleeAim(boss,attacker){
  if(attacker?.typeId!=='minecraft:player')return false;
  try {
    const origin=attacker.getHeadLocation(),dir=attacker.getViewDirection();
    const eye=eyeCenter(boss),f=flatView(boss),off=vec(origin,eye);
    if(off.x*f.x+off.z*f.z<-0.1)return false;
    const to=vec(eye,origin),t=to.x*dir.x+to.y*dir.y+to.z*dir.z;
    if(t<0 || t>6.8)return false;
    const near={x:origin.x+dir.x*t,y:origin.y+dir.y*t,z:origin.z+dir.z*t};
    return dist(near,eye)<=EYE_RADIUS;
  } catch{return false;}
}
function fx(dim,id,p){try{dim.spawnParticle(id,p);}catch{}}
function sound(dim,id,p){try{dim.playSound(id,p,{volume:1.15,pitch:0.6});}catch{}}
function playersNear(boss,range){
  const pos=boss.location;return world.getAllPlayers().filter(p=>{
    try{return valid(p) && p.dimension.id===boss.dimension.id && dist(p.location,pos)<=range && p.getGameMode()!=='creative' && p.getGameMode()!=='spectator';}
    catch{return false;}
  });
}
function ring(dim,point,r,num=18,id=DUST){
  for(let i=0;i<num;i++){
    const a=i*Math.PI*2/num;
    fx(dim,id,{x:point.x+Math.cos(a)*r,y:point.y+0.12,z:point.z+Math.sin(a)*r});
  }
}
function stomp(boss){
  if(!valid(boss)||isTimeFrozen(boss))return;
  playBossMotion(boss,'slam');
  const pos={...boss.location},dir=flatView(boss);
  const impact={x:pos.x+dir.x*1.4,y:pos.y,z:pos.z+dir.z*1.4};
  const dim=boss.dimension, radius=3.9;
  ring(dim,impact,radius,14,CRIT);
  sound(dim,'mob.irongolem.throw',pos);
  system.runTimeout(()=>{
    if(!valid(boss)||isTimeFrozen(boss))return;
    ring(dim,impact,radius,25,CRIT);
    ring(dim,impact,2.2,16,DUST);
    sound(dim,'random.explode',impact);
    for(const p of playersNear(boss,6.4)){
      const l=p.location,d=Math.hypot(l.x-impact.x,l.z-impact.z);
      if(d>radius || Math.abs(l.y-impact.y)>2.5)continue;
      try{p.applyDamage(18*levelAttackMultiplier(boss.getDynamicProperty?.(MONSTER_LEVEL_KEY)??1),{cause:EntityDamageCause.entityAttack,damagingEntity:boss});}catch{}
      try{
        const n=d||1;
        p.applyImpulse({x:(l.x-impact.x)/n*0.9,y:0.19,z:(l.z-impact.z)/n*0.9});
      }catch{}
      try{p.addEffect('slowness',40,{amplifier:1,showParticles:false});}catch{}
    }
  },16);
}
function bonusDamage(boss,attacker,damage){
  if(!valid(boss)||!Number.isFinite(damage)||damage<=0 || BONUS_IN_FLIGHT.has(boss.id))return;
  BONUS_IN_FLIGHT.add(boss.id);
  // 3x damage = vanilla hit once + a separately attributed 2x weak-spot bonus.
  try{boss.applyDamage(damage*2,{cause:EntityDamageCause.entityAttack,damagingEntity:attacker});}
  catch{}
  try{const p=eyeCenter(boss);for(let i=0;i<4;i++)fx(boss.dimension,CRIT,{x:p.x+(i%2?0.24:-0.24),y:p.y+(i%3-1)*0.17,z:p.z});}catch{}
  try{if(attacker?.typeId==='minecraft:player')attacker.onScreenDisplay?.setActionBar('§c외눈박이 거인 약점 명중! §e치명타 ×3');}catch{}
  system.runTimeout(()=>BONUS_IN_FLIGHT.delete(boss.id),2);
}
export function registerCyclopsBoss(){
  world.afterEvents.projectileHitEntity.subscribe(ev=>{
    try{
      const b=ev.getEntityHit()?.entity;
      if(b?.typeId!==CYCLOPS_ID || !ev.projectile?.id)return;
      if(isEyeProjectileHit(b,ev.location,ev.source))
        PROJECTILE_EYES.set(ev.projectile.id,{boss:b.id,source:ev.source,tick:system.currentTick});
    }catch{}
  });
  world.afterEvents.entityHurt.subscribe(ev=>{
    const b=ev.hurtEntity;
    if(b?.typeId!==CYCLOPS_ID||BONUS_IN_FLIGHT.has(b.id))return;
    const damage=ev.damage,src=ev.damageSource;
    if(!Number.isFinite(damage)||damage<=0)return;
    const projectileId=src?.damagingProjectile?.id;
    if(projectileId){
      // Bedrock emits the projectile-hit and hurt notifications in either order.
      system.runTimeout(()=>{
        const hit=PROJECTILE_EYES.get(projectileId);
        if(hit && hit.boss===b.id && system.currentTick-hit.tick<=6){
          PROJECTILE_EYES.delete(projectileId);
          bonusDamage(b,hit.source,damage);
        }
      },1);
      return;
    }
    const player=src?.damagingEntity;
    if(src?.cause===EntityDamageCause.entityAttack && isEyeMeleeAim(b,player))
      system.run(()=>bonusDamage(b,player,damage));
  });
  // Use only loaded bosses, and prune event/AI state to prevent world-save memory leaks.
  system.runInterval(()=>{
    const tick=system.currentTick, seen=new Set();
    for(const dimName of ['overworld','nether','the_end']){
      let dim,bosses;try{dim=world.getDimension(dimName);bosses=dim.getEntities({type:CYCLOPS_ID});}catch{continue;}
      for(const boss of bosses){
        if(!valid(boss)||isTimeFrozen(boss))continue;
        seen.add(boss.id);
        let st=STATE.get(boss.id);
        if(!st){st={next:tick+90};STATE.set(boss.id,st);}
        const players=playersNear(boss,7);
        if(!players.length)continue;
        const hp=boss.getComponent('minecraft:health');
        const fury=(hp?.currentValue??850)<(hp?.effectiveMax??850)*0.5;
        if(tick>=st.next && players.some(p=>dist(p.location,boss.location)<5.5)){
          st.next=tick+(fury?125:175);
          stomp(boss);
        }
      }
    }
    for(const id of STATE.keys())if(!seen.has(id))STATE.delete(id);
    for(const [id,hit] of PROJECTILE_EYES)if(tick-hit.tick>7)PROJECTILE_EYES.delete(id);
  },10);
}
```

## RPG_BP_v0578/scripts/core/golemBoss.js

```javascript
import {isTimeFrozen} from './timeStopDamage.js';
import {levelAttackMultiplier,MONSTER_LEVEL_KEY} from '../data/monsterLevels.js';
import {playBossMotion} from './bossMotion.js';
import { world, system, EntityDamageCause } from "@minecraft/server";

// 고대 마력 골렘: 실제 보스바와 450 XP 보상은 엔티티 컴포넌트로 처리.
const IDS = ["rpg:rock_golem_boss"];
const active = new Map();
const PARTICLE = "minecraft:critical_hit_emitter";
const DUST = "minecraft:basic_smoke_particle";

function okay(e) {
  try { return !!e?.isValid && (e.getComponent("minecraft:health")?.currentValue ?? 0) > 0; }
  catch { return false; }
}
function sameDimension(a, b) {
  try { return a.dimension.id === b.dimension.id; } catch { return false; }
}
function distance(a,b) {
  const dx=a.x-b.x, dy=a.y-b.y, dz=a.z-b.z;
  return Math.sqrt(dx*dx+dy*dy+dz*dz);
}
function playersNear(boss,range) {
  const center=boss.location;
  return world.getAllPlayers().filter(p=> okay(p) && sameDimension(p,boss) && distance(p.location,center)<=range &&
     p.getGameMode?.() !== "creative" && p.getGameMode?.() !== "spectator");
}
function safeParticle(dim, id, pos) {
  try { dim.spawnParticle(id,pos); } catch {}
}
function ring(dim,c,r,n=18,id=DUST) {
  for(let i=0;i<n;i++) {
    const angle=i*Math.PI*2/n;
    safeParticle(dim,id,{x:c.x+Math.cos(angle)*r,y:c.y+0.16,z:c.z+Math.sin(angle)*r});
  }
}
function damage(boss,target,amount,knock=0) {
  if(isTimeFrozen(boss)||!okay(target))return;
  try { target.applyDamage(amount*levelAttackMultiplier(boss.getDynamicProperty?.(MONSTER_LEVEL_KEY)??1),{cause:EntityDamageCause.entityAttack,damagingEntity:boss}); } catch {}
  if(knock>0) {
    try {
      const a=target.location,b=boss.location,dx=a.x-b.x,dz=a.z-b.z;
      const l=Math.hypot(dx,dz)||1;
      target.applyImpulse({x:dx/l*knock,y:0.28,z:dz/l*knock});
    } catch {}
  }
}
function groundSlam(boss) {
  if((!okay(boss)||isTimeFrozen(boss)))return;
  playBossMotion(boss,'slam');
  const dim=boss.dimension,loc={...boss.location}, radius=4.5;
  ring(dim,loc,radius,16,"minecraft:critical_hit_emitter");
  try{ dim.playSound("mob.irongolem.throw",loc,{volume:1.3,pitch:0.7}); }catch{}
  system.runTimeout(()=>{
    if((!okay(boss)||isTimeFrozen(boss)))return;
    const c=boss.location;
    ring(dim,c,radius,24,PARTICLE);
    for(const p of playersNear(boss,radius+0.6)) {
      damage(boss,p,11.25,1.1);
    }
    try{dim.playSound("random.explode",c,{volume:1.0,pitch:0.55});}catch{}
  },14);
}
function shockwave(boss) {
  if((!okay(boss)||isTimeFrozen(boss)))return;
  playBossMotion(boss,'wave');
  const dim=boss.dimension,origin={...boss.location}, touched=new Set();
  try{dim.playSound("mob.irongolem.hit",origin,{volume:1.1,pitch:0.6});}catch{}
  let step=0;
  const id=system.runInterval(()=>{
    if((!okay(boss)||isTimeFrozen(boss))||step>=10){system.clearRun(id);return;}
    step++;
    const r=step*0.9;
    ring(dim,origin,r,18,step%2===0?PARTICLE:DUST);
    for(const p of playersNear(boss,12)) {
      const d=Math.hypot(p.location.x-origin.x,p.location.z-origin.z);
      if(!touched.has(p.id)&&d>=r-1.1&&d<=r+1.0&&Math.abs(p.location.y-origin.y)<=3) {
        touched.add(p.id);damage(boss,p,6.75,0.55);
      }
    }
  },2);
}
export function registerGolemBoss() {
  const dimensions=["overworld","nether","the_end"];
  system.runInterval(()=>{
    const now=system.currentTick;
    const seen=new Set();
    for(const id of dimensions) {
      let dim;
      try { dim=world.getDimension(id); } catch { continue; }
      let bosses;
      try { bosses=IDS.flatMap(type => dim.getEntities({type})); } catch { continue; }
      for(const boss of bosses) {
        if((!okay(boss)||isTimeFrozen(boss)))continue;
        seen.add(boss.id);
        let state=active.get(boss.id);
        if(!state) { state={slam:now+100,wave:now+180};active.set(boss.id,state); }
        const hp=boss.getComponent("minecraft:health");
        const enraged=(hp?.currentValue??600) <= (hp?.effectiveMax??600)*0.5;
        const near=playersNear(boss,16);
        if(!near.length)continue;
        if(now>=state.slam&&near.some(p=>distance(p.location,boss.location)<5.4)) {
          state.slam=now+(enraged?115:165);
          groundSlam(boss);
        }
        if(now>=state.wave) {
          state.wave=now+(enraged?220:320);
          shockwave(boss);
        }
        if(enraged&&now%40<10) safeParticle(dim,PARTICLE,{x:boss.location.x,y:boss.location.y+2.0,z:boss.location.z});
      }
    }
    for(const id of [...active.keys()])if(!seen.has(id))active.delete(id);
  },10);
}
```

## RPG_BP_v0578/scripts/core/input.js

```javascript
import {RUNNERS as WORLD_RUNNERS} from '../skills/theWorld.js';
import {isTimeFrozen} from './timeStopDamage.js';
import {useProgressionItem} from './progressionItems.js';
import {RUNNERS as WAR_GOD_RUNNERS} from '../skills/warGod.js';
import { executeChameleonSkill,skillBookItemUse,SKILLBOOK_ID } from './chameleonProgress.js';
import { world, system } from "@minecraft/server";
import { CONFIG } from "../config.js";
import { areAllies } from "./social.js";
import { SKILLS } from "../data/skills.js";
import { RUNNERS as SWORD_RUNNERS, fireBasic } from "../skills/sword.js";
import { RUNNERS as FLAME_SPIN_RUNNERS } from "../skills/swordFlameSpin.js";
import { RUNNERS as MAGE_RUNNERS } from "../skills/mage.js";
import { RUNNERS as HOLY_RUNNERS } from "../skills/holy.js";
import { RUNNERS as FIST_RUNNERS } from "../skills/fist.js";
import { RUNNERS as ARCHER_RUNNERS, shootHeavenlyInstant, clearPreparedBowMode,heavenlyAmmoCost } from "../skills/archer.js";
import { RUNNERS as NECRO_RUNNERS } from "../skills/necro.js";
import { RUNNERS as DARK_RUNNERS } from "../skills/dark.js";
import { RUNNERS as BERSERKER_RUNNERS } from "../skills/berserker.js";
import { RUNNERS as BLOOD_RUNNERS, bloodAwakenIsActive, bloodAwakenStop } from "../skills/blood.js";
import { RUNNERS as HACKER_RUNNERS, isHackerStealthActive, stopHackerStealth } from "../skills/hacker.js";
import { RUNNERS as HARPOON_RUNNERS } from "../skills/harpoon.js";
import { RUNNERS as FROST_RUNNERS } from "../skills/frost.js";
import { RUNNERS as FIRE_RUNNERS } from "../skills/fire.js";
import { RUNNERS as ASSASSIN_RUNNERS, assassinStealthIsActive, stopAssassinStealth } from "../skills/assassin.js";
import { RUNNERS as TECH_RUNNERS } from "../skills/technomancer.js";
import { RUNNERS as DEMON_RUNNERS } from "../skills/demon.js";
import { RUNNERS as CELESTIAL_RUNNERS, throwMoonWeapon, tryBorrowedMoonThrow } from "../skills/celestial.js";
import { RUNNERS as DRAGON_RUNNERS } from "../skills/dragon.js";
import { chameleonCastJob, swapChameleonWeapon } from "./chameleon.js";
import { openAdminTool, ADMIN_TOOL_ID } from "./adminTool.js";
import { RUNNERS as CHAMELEON_RUNNERS, registerChameleonDash } from "../skills/chameleonDash.js";
import {
  getIndex, setIndex, getMana, spendMana, cooldownLeft, startCooldown,
} from "./playerData.js";
import { sound, particle, heldItemId } from "./util.js";
import { manaCostFor } from "./stats.js";
import { getContext } from "./context.js";
import { notify, refreshHud } from "./hud.js";
import { openSkillMenu } from "./skillMenu.js";

const RUNNERS = { ...WORLD_RUNNERS, ...WAR_GOD_RUNNERS, ...SWORD_RUNNERS, ...FLAME_SPIN_RUNNERS, ...MAGE_RUNNERS, ...HOLY_RUNNERS, ...FIST_RUNNERS, ...ARCHER_RUNNERS, ...NECRO_RUNNERS, ...DARK_RUNNERS, ...BERSERKER_RUNNERS, ...BLOOD_RUNNERS, ...HACKER_RUNNERS, ...HARPOON_RUNNERS, ...FROST_RUNNERS, ...FIRE_RUNNERS, ...ASSASSIN_RUNNERS, ...TECH_RUNNERS, ...DEMON_RUNNERS, ...CELESTIAL_RUNNERS, ...DRAGON_RUNNERS, ...CHAMELEON_RUNNERS };
const isPlayer = (e) => e?.typeId === "minecraft:player";

const CHAMELEON_SWORD = "rpg:chameleon_flame_blade";
const CHAMELEON_SCYTHE = "rpg:chameleon_moon_scythe";
const CHAMELEON_SWING_CRIMSON = "rpg:chameleon_crimson_swing";
const CHAMELEON_SWING_PLATINUM = "rpg:chameleon_platinum_swing";
const swingFxTick = new Map();

function emitChameleonSwing(player) {
  try {
    if (!isPlayer(player)) return;
    const held = heldItemId(player);
    const fx = held === CHAMELEON_SWORD ? CHAMELEON_SWING_CRIMSON
      : held === CHAMELEON_SCYTHE ? CHAMELEON_SWING_PLATINUM : undefined;
    if (!fx) return;
    const now = system.currentTick;
    if (now - (swingFxTick.get(player.id) ?? -100) < 3) return;
    swingFxTick.set(player.id, now);
    // Three small, spinning slash billboards sweep in an arc in front of the hand.
    // Keep them visual-only: they do not alter melee damage or skill mechanics.
    const dim = player.dimension;
    for (let phase = 0; phase < 3; phase++) {
      system.runTimeout(() => {
        try {
          if (!player.isValid || player.dimension.id !== dim.id || heldItemId(player) !== held) return;
          const head = player.getHeadLocation();
          const view = player.getViewDirection();
          const length = Math.hypot(view.x, view.z) || 1;
          const front = { x: view.x / length, z: view.z / length };
          const right = { x: -front.z, z: front.x };
          const side = (phase - 1) * 0.42;
          const pos = {
            x: head.x + front.x * (0.95 + 0.08 * phase) + right.x * side,
            y: head.y - 0.42 + (phase - 1) * 0.22,
            z: head.z + front.z * (0.95 + 0.08 * phase) + right.z * side,
          };
          particle(dim, fx, pos);
        } catch { /* player unavailable */ }
      }, phase);
    }
  } catch {}
}

// 아군에게 발동 가능한 왼쪽 클릭 기술만 허용. 동일한 공격이
// beforeEvents.entityHurt + afterEvents.entityHitEntity 양쪽에서 관측되더라도
// 한 번만 발동하도록 2틱 중복 방지.
const friendlyHitTick = new Map();
export function handleFriendlyHit(player, target) {
  if (!isPlayer(player) || !isPlayer(target) || player.id === target.id ||
      (!areAllies(player,target)&&CONFIG.pvp)) return;
  const ctx = getContext(player);
  if (!ctx?.skills?.length) return;
  const def = SKILLS[ctx.skills[getIndex(player) % ctx.skills.length]];
  if (!def || !(def.id === "hacker_hack" || def.id === "holy_heal" || def.id === "holy_orb")) return;
  if(!areAllies(player,target)&&def.id!=="holy_heal")return;
  const key = `${player.id}:${target.id}`;
  const now = system.currentTick;
  if (now - (friendlyHitTick.get(key) ?? -100) < 3) return;
  friendlyHitTick.set(key, now);
  cast(player, ctx, target);
}

// 우클릭: 현재 선택된 스킬 발동
// 좌클릭으로 엔티티를 때려서 쓰는 스킬(trigger: "hit")은 때린 대상(target)이 있을 때만 발동합니다.
function cast(player, ctx, target) {
  if(isTimeFrozen(player))return;
  const list = ctx.skills;
  if (list.length === 0) return;

  const def = SKILLS[list[getIndex(player) % list.length]];
  const runner = RUNNERS[def.id];
  if (!runner) return;
  if(!CONFIG.pvp&&isPlayer(target)&&!(["holy_heal","holy_orb","hacker_hack"].includes(def.id)))return;

  const isHit = def.trigger === "hit";
  // 해킹: 적에게는 우클릭 해킹, 같은 길드/파티원에게는 좌클릭 지원.
  const allyHack = def.id === "hacker_hack" && isPlayer(target) &&
    target.id !== player.id && areAllies(player, target);
  if (isHit && !target) {
    notify(player, `§7${def.name}: 엔티티를 좌클릭해서 사용합니다.`);
    return;
  }
  // 계약은 우클릭 자가 강화와 좌클릭 적 대상 효과를 모두 사용합니다.
  if (!isHit && target && !allyHack && def.id !== "demon_contract" && def.id !== "fallen_orb") return;

  // 해제 입력은 쿨타임·마나를 쓰지 않고 바로 상태를 해제합니다.
  if (def.id === "blood_awaken" && bloodAwakenIsActive(player)) {
    bloodAwakenStop(player); refreshHud(player, true); return;
  }
  if (def.id === "hacker_stealth" && isHackerStealthActive(player)) {
    stopHackerStealth(player); refreshHud(player, true); return;
  }
  if (def.id === "assassin_stealth" && assassinStealthIsActive(player)) {
    stopAssassinStealth(player); refreshHud(player, true); return;
  }
  if(def.id === "moon_chakram_split" && tryBorrowedMoonThrow(player)){refreshHud(player,true);return;}
  const left = cooldownLeft(player, def.id);
  if (left > 0) {
    notify(player, `§c${def.name} 쿨타임 ${(left / 20).toFixed(1)}초`);
    return;
  }
  const adjustedMana = def.id==='world_stop'?Math.max(300,manaCostFor(player,def.mana)):manaCostFor(player, def.mana);
  if (getMana(player) < adjustedMana) {
    notify(player, "§c마나가 부족합니다.");
    return;
  }

  const charged=executeChameleonSkill(player,def.id,()=>runner(player,chameleonCastJob(def.id,ctx.job),def,target));
  if(!charged.ok){notify(player,'§c'+charged.message);return;}
  const castResult=charged.result;
  if (castResult === false) return;

  spendMana(player, adjustedMana);
  // 복수귀 처형은 성공과 실패의 재사용 대기시간이 다릅니다.
  startCooldown(player, def.id, castResult && typeof castResult.cooldown === "number" ? castResult.cooldown : def.cooldown);
  refreshHud(player, true);
}

// 휘두르기: 푸른 에너지 (3차 이상 검 계열)
function onSwing(player) {
  emitChameleonSwing(player);
  const ctx = getContext(player);
  if (!ctx?.job.basic || ctx.job.id === "apprentice") return;
  if (cooldownLeft(player, "basic") > 0) return;
  startCooldown(player, "basic", CONFIG.basic.cooldown);
  fireBasic(player, ctx.job);
}

// 웅크리기: 스킬 전환 (설정창을 열려고 웅크린 경우를 되돌릴 수 있게 직전 번호를 기억)
const sneakLog = new Map(); // playerId -> { tick, prev }

function onSneakStart(player) {
  const ctx = getContext(player);
  if (!ctx || ctx.skills.length === 0) return;
  sneakLog.set(player.id, { tick: system.currentTick, prev: getIndex(player) });
  setIndex(player, (getIndex(player) + 1) % ctx.skills.length);
  sound(player.dimension, "random.click", player.location, { volume: 0.6, pitch: 1.3 });
  refreshHud(player, true);
}

// Instant heavenly bow shoots at fully charged velocity. Limit successful
// shots to one per 10 ticks (0.5 sec), also preventing duplicate item events.
const instantShotTick = new Map();
const gadgetCastTick = new Map();
function gadgetClick(p) {
  if (!isPlayer(p)) return;
  const ctx=getContext(p);
  if (!ctx || !['blood','hacker','harpoon','frost','fire','assassin','tech','demon','dragon','celestial_sun','celestial_moon','chameleon','war_god','the_world'].includes(ctx.line)) return;
  const now=system.currentTick;
  if(now-(gadgetCastTick.get(p.id)??-100)<3)return;
  gadgetCastTick.set(p.id,now);
  try {
    // 카멜레온: 웅크리고 무기 우클릭하면 즉시 장비 전환.
    if (ctx.job.id === 'chameleon' && p.isSneaking) {
      const log=sneakLog.get(p.id);
      if(log && now-log.tick<=20)setIndex(p,log.prev);
      const result=swapChameleonWeapon(p);
      notify(p,result.message);
      return;
    }
    if(CONFIG.skillMenuOnSneakUse && p.isSneaking){
      const log=sneakLog.get(p.id);
      if(log && now-log.tick<=20)setIndex(p,log.prev);
      openSkillMenu(p);
      return;
    }
  } catch {}
  cast(p,ctx);
}
function heavenlyClick(p) {
  if (!isPlayer(p)) return;
  const ctx=getContext(p);
  if(!ctx||ctx.job.id!=="heavenly_archer"||ctx.held!=="rpg:heavenly_bow")return;
  const now=system.currentTick;
  if(now-(instantShotTick.get(p.id)??-100)<10)return;
  try{
    if(CONFIG.skillMenuOnSneakUse && p.isSneaking){
      const log=sneakLog.get(p.id);
      if(log&&now-log.tick<=20)setIndex(p,log.prev);
      openSkillMenu(p);
      return;
    }
  }catch{}
  const basicAmmo=heavenlyAmmoCost(p);
  if(getMana(p)<basicAmmo){notify(p,'§c화살이 없습니다. 마나 화살 1발에 MP 20이 필요합니다.');return;}
  const def=SKILLS[ctx.skills[getIndex(p)%ctx.skills.length]];
  // Reserve the entire volley and skill cost before the runner spawns extras.
  // With too little MP for the selected skill, a basic 20-MP shot still works.
  const volleyCost=heavenlyAmmoCost(p,def?.id==='archer_rapid'?5:1);
  const canCast=def&&getMana(p)>=volleyCost+manaCostFor(p,def.mana);
  clearPreparedBowMode(p);
  if(canCast)cast(p,ctx);
  if(shootHeavenlyInstant(p))instantShotTick.set(p.id,now);
  else notify(p,"§c화살을 발사하지 못했습니다.");
}

// Custom item components must be registered during startup. Each is isolated:
// an unsupported/duplicate item component must never interrupt RPG commands.
system.beforeEvents.startup.subscribe(e=>{
  const registry = e?.itemComponentRegistry;
  if (!registry || typeof registry.registerCustomComponent !== "function") {
    console.warn("[RPG] ItemComponentRegistry unavailable; command registration remains active");
    return;
  }
  const safe = (name, handler) => {
    try { registry.registerCustomComponent(name, handler); }
    catch (error) { console.warn("[RPG] item component " + name + ": " + String(error)); }
  };
  safe("rpg:progression_item",{onUse(e){system.run(()=>useProgressionItem(e.source,e.itemStack?.typeId??heldItemId(e.source)));}});
  safe("rpg:random_skillbook",{onUse(e){system.run(()=>skillBookItemUse(e.source,system.currentTick));}});
  safe("rpg:throw_moon_chakram", {
    onUse(e) { system.run(() => throwMoonWeapon(e.source)); },
  });
  safe("rpg:cast_item",{
    onUse(e){system.run(()=>gadgetClick(e.source));},
  });
  safe("rpg:admin_settings", {
    onUse(e) { system.run(() => openAdminTool(e.source)); },
  });
  safe("rpg:instant_bow",{
    onUse(e){system.run(()=>heavenlyClick(e.source));},
    onCompleteUse(e){system.run(()=>heavenlyClick(e.source));},
  });
});

export function registerInput() {
  registerChameleonDash();
  // 커스텀 활을 당겼다가 놓았을 때 일반 화살 사격 + 현재 궁수 스킬을 사용합니다.
  // shooter 발사와 별개로 스킬 코드는 같은 입력으로 한 번만 실행됩니다.
  world.afterEvents.itemReleaseUse?.subscribe((ev) => {
    const p = ev.source;
    if (!isPlayer(p)) return;
    const ctx = getContext(p);
    if (!ctx || ctx.line !== "archer" || ev.itemStack?.typeId !== ctx.held) return;
    if(ctx.job.id==="heavenly_archer")return; // instant bow is handled on use, not release
    try {
      if (p.isSneaking) { openSkillMenu(p); return; }
    } catch { /* 사용 가능 */ }
    cast(p, ctx);
  });

  world.afterEvents.itemUse.subscribe((ev) => {
    const p = ev.source;
    if (!isPlayer(p)) return;
    if(ev.itemStack?.typeId===SKILLBOOK_ID){system.run(()=>skillBookItemUse(p,system.currentTick));return;}
    if (ev.itemStack?.typeId === ADMIN_TOOL_ID) { openAdminTool(p); return; }
    const ctx = getContext(p);
    if (!ctx) return;
    if(ctx.line === 'blood' || ctx.line === 'hacker' || ctx.line === 'harpoon' || ctx.line === 'frost' || ctx.line === 'fire' || ctx.line === 'assassin' || ctx.line === 'demon' || ctx.line === 'dragon' || ctx.line === 'celestial_sun' || ctx.line === 'celestial_moon' || ctx.line === 'chameleon' || ctx.line === 'war_god' || ctx.line === 'the_world'){
      gadgetClick(p);return;
    }
    if (ctx.line === "archer") {
      // Item event fallback if the platform forwards item use but not custom onUse.
      if(ctx.job.id==="heavenly_archer"&&ev.itemStack?.typeId==="rpg:heavenly_bow")heavenlyClick(p);
      return;
    }

    // 웅크린 채로 우클릭: 스킬 설정창 (발동하지 않음)
    let sneaking = false;
    try {
      sneaking = p.isSneaking;
    } catch {
      // 상태를 읽을 수 없으면 일반 발동
    }
    if (CONFIG.skillMenuOnSneakUse && sneaking) {
      // 방금 웅크리면서 전환된 스킬 번호는 원래대로 돌려놓습니다.
      const log = sneakLog.get(p.id);
      if (log && system.currentTick - log.tick <= 20) setIndex(p, log.prev);
      openSkillMenu(p);
      return;
    }
    cast(p, ctx);
  });

  // 허공을 휘두르는 동작은 버전에 따라 이벤트 지원이 달라서, 있을 때만 구독합니다.
  world.afterEvents.playerLeave.subscribe((ev) => {
    sneakLog.delete(ev.playerId);instantShotTick.delete(ev.playerId);gadgetCastTick.delete(ev.playerId);swingFxTick.delete(ev.playerId);
    for (const key of friendlyHitTick.keys()) if (key.startsWith(`${ev.playerId}:`) || key.endsWith(`:${ev.playerId}`)) friendlyHitTick.delete(key);
  });

  world.afterEvents.playerSwingStart?.subscribe((ev) => {
    onSwing(ev.player);
    try{
      const hits=ev.player.getEntitiesFromViewDirection({maxDistance:4,ignoreBlockCollision:false}).sort((a,b)=>a.distance-b.distance);
      if(isPlayer(hits[0]?.entity)&&(areAllies(ev.player,hits[0].entity)||!CONFIG.pvp))handleFriendlyHit(ev.player,hits[0].entity);
    }catch{}
  });

  // 대상/블록을 맞춘 휘두르기는 항상 감지됩니다. (쿨타임으로 중복 발동 방지)
  world.afterEvents.entityHitEntity.subscribe((ev) => {
    const p = ev.damagingEntity;
    if (!isPlayer(p)) return;
    onSwing(p);
    // 좌클릭으로 때려서 쓰는 스킬 (힐 / 축복의 구슬 / Sacrifice)
    const ctx = getContext(p);
    if (ctx && ev.hitEntity) {
      if (isPlayer(ev.hitEntity) && (areAllies(p, ev.hitEntity)||!CONFIG.pvp)) handleFriendlyHit(p, ev.hitEntity);
      else cast(p, ctx, ev.hitEntity);
    }
  });
  world.afterEvents.entityHitBlock.subscribe((ev) => {
    if (isPlayer(ev.damagingEntity)) onSwing(ev.damagingEntity);
  });

  // 웅크리기 시작 순간 감지
  const wasSneaking = new Map();
  system.runInterval(() => {
    for (const p of world.getAllPlayers()) {
      let now = false;
      try {
        now = p.isSneaking;
      } catch {
        continue;
      }
      if (now && !wasSneaking.get(p.id)) onSneakStart(p);
      wasSneaking.set(p.id, now);
    }
  }, 1);
}
```

## RPG_BP_v0578/scripts/core/playerData.js

```javascript
import { system } from "@minecraft/server";
import { CONFIG } from "../config.js";
import { JOBS } from "../data/jobs.js";

// 월드를 다시 열어도 유지되는 데이터는 플레이어 동적 속성에 저장합니다.
const KEY = {
  job: "rpg:job",
  skills: "rpg:skills", // 배운 스킬 id 배열(JSON)
  mana: "rpg:mana",
  forcedManaMax: "rpg:admin_mana_max",
  index: "rpg:skill_index",
  progress: "rpg:progress", // 전직 조건 진행도(JSON)
  disabled: "rpg:disabled_skills", // 사용자가 꺼 둔 스킬 id 배열(JSON)
};

export function getJobId(player) {
  const v = player.getDynamicProperty(KEY.job);
  // 과거 버전에서 미구현 반인반수 직업을 강제로 지정했던 저장값은 견습으로 취급.
  if (v === "beastkin") return "apprentice";
  return typeof v === "string" && JOBS[v] ? v : undefined;
}

export function setJobId(player, id) {
  player.setDynamicProperty(KEY.job, id);
}

export function getSkills(player) {
  try {
    const v = player.getDynamicProperty(KEY.skills);
    const arr = typeof v === "string" ? JSON.parse(v) : [];
    return Array.isArray(arr) ? arr : [];
  } catch {
    return [];
  }
}

export function setSkills(player, arr) {
  player.setDynamicProperty(KEY.skills, JSON.stringify(arr));
}

// ---- 꺼 둔 스킬 (스킬 설정창에서 변경) ----
export function getDisabled(player) {
  try {
    const v = player.getDynamicProperty(KEY.disabled);
    const arr = typeof v === "string" ? JSON.parse(v) : [];
    return Array.isArray(arr) ? arr : [];
  } catch {
    return [];
  }
}

export function setDisabled(player, arr) {
  player.setDynamicProperty(KEY.disabled, JSON.stringify(arr));
}

// ---- 전직 조건 진행도 ----
export function getProgress(player) {
  try {
    const v = player.getDynamicProperty(KEY.progress);
    const o = typeof v === "string" ? JSON.parse(v) : {};
    return o && typeof o === "object" ? o : {};
  } catch {
    return {};
  }
}

export function addProgress(player, key, n = 1) {
  const o = getProgress(player);
  o[key] = (o[key] ?? 0) + n;
  player.setDynamicProperty(KEY.progress, JSON.stringify(o));
  return o[key];
}

// ---- 마나 ----
// 최대 마나 = 기본 200 + (바친 책 수 x 권당 증가량), 직업별 상한 안에서.
// 책 수로 계산하기 때문에 견습 때 바친 책도 마법사로 전직하면 권당 20으로 다시 계산됩니다.
export function isHighMana(player) {
  return !!JOBS[getJobId(player)]?.highMana;
}

// 최대 마나
//  - 마법사/성직자 계열: 200 + 바친 책 수 x 20, 상한 없음 (책을 바치는 만큼 계속 늘어남)
//  - 그 외 직업: 항상 normalMax (1000)
export function maxManaOf(player) {
  // 관리자 전용 강제 마나 저장값은 모든 직업의 기본 최대 마나보다 우선한다.
  try {
    const forced=player.getDynamicProperty(KEY.forcedManaMax);
    if(Number.isSafeInteger(forced) && forced>=1 && forced<=100000)return forced;
  }catch{}
  if (!isHighMana(player)) return CONFIG.mana.normalMax;
  const books = getProgress(player).books ?? 0;
  return CONFIG.mana.base + books * CONFIG.mana.perBookHigh;
}

// 관리자 명령어는 현재 마나뿐 아니라 '최대 마나'를 저장하고 완전히 채운다.
// reset 인자는 강제 설정을 제거하고 원래 직업/책의 마나 공식으로 복원한다.
export function forceSetManaMax(player,input){
  const value=String(input??'').trim().toLowerCase();
  if(value==='reset'){
    try{
      player.setDynamicProperty(KEY.forcedManaMax,undefined);
      setMana(player,maxManaOf(player));
      const max=maxManaOf(player),current=getMana(player);
      if(current!==max)return {ok:false,reason:'마나 초기화 후 값 확인 실패'};
      return {ok:true,max,current,reset:true};
    }catch(error){return {ok:false,reason:'마나 설정 초기화 실패: '+String(error)};}
  }
  const amount=/^[0-9]+$/.test(value)?Number(value):NaN;
  if(!Number.isSafeInteger(amount)||amount<1||amount>100000)
    return {ok:false,reason:'강제 마나는 1~100,000 정수 또는 reset만 가능합니다.'};
  try{
    player.setDynamicProperty(KEY.forcedManaMax,amount);
    setMana(player,amount);
    const max=maxManaOf(player),current=getMana(player);
    if(max!==amount||current!==amount)return {ok:false,reason:`마나 저장 확인 실패 (현재 ${current}/${max})`};
    return {ok:true,max,current,reset:false};
  }catch(error){return {ok:false,reason:'마나 저장 실패: '+String(error)};}
}
// 1000 이상은 1k, 1.5k, 12k, 3.2M 처럼 줄여서 표시
export function formatMana(n) {
  n = Math.floor(n);
  const units = [[1e12, "T"], [1e9, "B"], [1e6, "M"], [1e3, "k"]];
  for (const [v, s] of units) {
    if (n >= v) {
      const x = n / v;
      const t = x >= 100 ? Math.floor(x).toString() : (Math.floor(x * 10) / 10).toString();
      return t + s;
    }
  }
  return String(n);
}
export function manaText(player) {
  return formatMana(getMana(player));
}
export function maxManaText(player) {
  return formatMana(maxManaOf(player));
}

export function getMana(player) {
  const v = player.getDynamicProperty(KEY.mana);
  const max = maxManaOf(player);
  return typeof v === "number" ? Math.min(v, max) : max;
}

export function setMana(player, value) {
  player.setDynamicProperty(KEY.mana, Math.max(0, Math.min(maxManaOf(player), value)));
}

// 스킬 사용 시 마나 소모. 마법사 상태에서 쓴 마나는 현자 전직 조건에 누적됩니다.
export function spendMana(player, amount) {
  setMana(player, getMana(player) - amount);
  if (["mage", "cleric"].includes(getJobId(player))) addProgress(player, "mana_spent", amount);
}

export function getIndex(player) {
  const v = player.getDynamicProperty(KEY.index);
  return typeof v === "number" ? v : 0;
}

export function setIndex(player, i) {
  player.setDynamicProperty(KEY.index, i);
}

// ---- 쿨타임: 온라인 틱 + 재접속/서버 재시작용 만료 시각 저장 ----
const cooldowns = new Map(); // playerId -> Map(skillId -> 끝나는 틱)
export const COOLDOWN_KEY='rpg:cooldowns_v1';
function cooldownMap(player){
  if(cooldowns.has(player.id))return cooldowns.get(player.id);
  const m=new Map(),raw=player.getDynamicProperty(COOLDOWN_KEY);
  if(raw!==undefined){
    const saved=JSON.parse(raw);if(saved.version!==1||!saved.until||typeof saved.until!=='object')throw Error('쿨타임 저장 데이터 오류');
    const now=Date.now();
    for(const [id,end]of Object.entries(saved.until)){
      if(!Number.isSafeInteger(end)||end<0)throw Error('쿨타임 저장 시각 오류');
      const left=Math.ceil((end-now)/50);if(left>0)m.set(id,system.currentTick+left);
    }
  }
  cooldowns.set(player.id,m);return m;
}
function saveCooldowns(player,m){
  const now=Date.now(),until={};
  for(const [id,tick]of m){const left=tick-system.currentTick;if(left>0)until[id]=now+Math.ceil(left*50);}
  player.setDynamicProperty(COOLDOWN_KEY,JSON.stringify({version:1,until}));
}

export function cooldownLeft(player, key) {
  const until = cooldownMap(player).get(key) ?? 0;
  return Math.max(0, until - system.currentTick);
}

export function startCooldown(player, key, ticks) {
  if(!Number.isFinite(ticks)||ticks<0)throw Error('잘못된 쿨타임');
  const m=cooldownMap(player);
  m.set(key, system.currentTick + ticks);
  saveCooldowns(player,m);
}

// 궁극기만 제외하고 스킬 쿨타임을 초기화 (해커 시스템 재부팅).
const ULTIMATE_IDS = new Set(["world_stop","war_god_black_flash","sword_rush","fist_finale","archer_heavenly","fallen_end","sage_zoltraak","blood_bind","hacker_emp","tech_storm","revenant_execution","frost_absolute","fire_emperor","shadow_assassinate","beast_grand_explosion"]);
export function resetNonUltimateCooldowns(player) {
  const m=cooldownMap(player);
  for (const id of [...m.keys()]) if (!ULTIMATE_IDS.has(id)) m.delete(id);
  saveCooldowns(player,m);
}

// 모든 스킬 쿨타임(기본 공격 포함) 초기화
export function resetCooldowns(player) {
  player.setDynamicProperty(COOLDOWN_KEY,undefined);
  cooldowns.delete(player.id);
}

export function clearPlayerRuntime(playerId) {
  cooldowns.delete(playerId);
}
```

## RPG_BP_v0578/scripts/core/rpgActions.js

```javascript
import {openTpa,openTpaInbox} from './tpa.js';
import {createPvpArena} from './pvpArena.js';
import {configurePeace} from './peaceZones.js';
import {openStatusTablet} from './statusTablet.js';
import {openFusionMenu} from './fusion.js';
import {openPvp,clearPkPenalty} from './pvp.js';
import {spawnGptNpc} from './gptNpc.js';
import {createTemple,removeTemple,openTemple} from './temples.js';
import {openArmorTransfer} from './armorTransfer.js';
import {askRpg} from './rpgAssistant.js';
import {createDungeon,enterDungeon,startDungeon,soloDungeon,leaveDungeon,openDungeons,removeDungeon,flushRewards} from './dungeons.js';
import {readAngelHealing} from './angelHealing.js';
import {claimDungeonLoot} from './dungeonRewards.js';
import { openRuneMenu } from './runeMenu.js';
import { openJobMenu } from './jobMenu.js';
import { openSkillGuide } from './skillGuide.js';
import { openEggMenu } from "./eggMenu.js";
import { openBeginnerGuide } from './beginner.js';
import { openEquipmentMenu } from './equipment.js';
import { openUpgradeMenu } from './upgrades.js';
import { openCraftingGuide } from './crafting.js';
import { openWalletMenu } from './wallet.js';
import { giveHitoEgg,openHitoEggMenu } from "./hito.js";
import { giveAdminTool } from "./adminTool.js";
import { CONFIG } from "../config.js";
import { giveRulebook } from "./rulebook.js";
import { openStatsMenu, statsSummary, forceSetLevel, getLevel } from "./stats.js";
import { openChameleonSkillbook, learnChameleonSkill, swapChameleonWeapon } from "./chameleon.js";
import { JOBS, findJob } from "../data/jobs.js";
import { SKILLS } from "../data/skills.js";
import {
  getJobId, getSkills, getMana, setMana, maxManaOf, manaText, maxManaText, getProgress, getDisabled, resetCooldowns, forceSetManaMax,
} from "./playerData.js";
import {
  requestAdvance, resetToApprentice, giveStarter, requirementStatus, giveAllItems,
} from "./advance.js";
import { offerItems, offerLight, craftHolyWater } from "./offer.js";
import { openSkillMenu } from "./skillMenu.js";
import { refreshHud } from "./hud.js";
import { toggleWings, inspectWings, refreshWings } from "./wings.js";
import { getFistCoatingStatus } from "./fistCoating.js";
import { openGuildMenu, openPartyMenu, openInvites } from "./social.js";

// 슬래시 명령어(/rpg:...)와 /scriptevent rpg:... 가 같은 동작을 하도록 한 곳에 모았습니다.
export const HELP =
  "§e사용법: /rpg:pvp [플레이어|accept|surrender] | /rpg:dungeon [1~10] | /rpg:dungeonstart | /rpg:dungeonsolo <1~10> | /rpg:dungeonleave | /rpg:dungeonloot | (관리자) /rpg:dungeoncreate <1~10|all> | /rpg:dungeonremove <이름> | /rpg:temple | (관리자) /rpg:templecreate | /rpg:templeremove | /rpg:transfer | /rpg:fuse | /rpg:ask [질문] | (관리자) /rpg:gptnpc | /rpg:skillguide | /rpg:runes | /rpg:jobs | /rpg:craft | /rpg:wallet | /rpg:upgrade | /rpg:equipment | /rpg:guide | /rpg:rulebook | /rpg:ping | /rpg:menu | /rpg:job <직업> | /rpg:status | /rpg:stats | /rpg:skills | /rpg:skillbook | /rpg:learn <ID> | /rpg:swap | /rpg:offer | /rpg:bless | /rpg:holywater | " +
  "/rpg:guild | /rpg:party | /rpg:invites | " +
  "(관리자) /rpg:admin (설정 도구) | /rpg:setlevel <1~100000> | /rpg:setmana <1~100000|reset> | /rpg:force <직업> | /rpg:reset | /rpg:starter <sword|mage|holy|fist|archer> | /rpg:items | /rpg:charge | /rpg:wings | /rpg:wingcheck | /rpg:wingrefresh | /rpg:fistcheck | /rpg:boss | /rpg:cyclops | /rpg:hydra | /rpg:techgolem";

// 공식 Player 권한 속성 사용. 2.0.0 환경에서 isOp() 메서드에 의존하지 않는다.
function isAdminPlayer(p){
  try { if (p.playerPermissionLevel === 2) return true; } catch {}
  try { if (typeof p.commandPermissionLevel === 'number' && p.commandPermissionLevel >= 1) return true; } catch {}
  try { if (typeof p.isOp === 'function' && p.isOp()) return true; } catch {}
  return false;
}

export function runRpgAction(action, p, arg = "") {
  const text = String(arg ?? "").trim();
  // /scriptevent / !rpg 경로에도 등록 명령어와 동일한 관리자 권한을 적용.
  const adminOnly = new Set(["gptnpc","force", "reset", "starter", "items", "egg", "charge", "boss", "cyclops", "hydra", "techgolem", "hito", "setlevel", "setmana", "admin", "dungeoncreate", "dungeonremove", "templecreate", "templeremove"]);
  if (adminOnly.has(action) && !isAdminPlayer(p)) {
    p.sendMessage("§c[RPG] 이 명령어는 관리자만 사용할 수 있습니다.");
    return;
  }
  switch (action) {
    case 'tpa':return openTpa(p);
    case 'tpaaccept':return openTpaInbox(p);
    case 'pvpcreate':return createPvpArena(p);
    case 'pkclear':return clearPkPenalty(p,text);
    case 'peace':return configurePeace(p,text);
    case 'pvp':return openPvp(p,text);
    case 'gptnpc':return spawnGptNpc(p);
    case 'temple': return openTemple(p);
    case 'templecreate': try{return createTemple(p);}catch(error){p.sendMessage('§e'+String(error.message??error));return;}
    case 'templeremove': try{return removeTemple(p);}catch(error){p.sendMessage('§e'+String(error.message??error));return;}
    case 'fuse': return openFusionMenu(p);
    case 'transfer': return openArmorTransfer(p);
    case 'ask': return askRpg(p,text);
    case 'dungeon':
    case 'dungeonstart':
    case 'dungeonsolo':
    case 'dungeoncreate':
    case 'dungeonremove':
    case 'dungeonleave':
    case 'dungeonloot': {
      try {
        if(action==='dungeonstart')startDungeon(p);
        else if(action==='dungeonsolo')soloDungeon(p,text);
        else if(action==='dungeoncreate')createDungeon(p,text);
        else if(action==='dungeonremove')removeDungeon(p,text);
        else if(action==='dungeonleave')leaveDungeon(p);
        else if(action==='dungeonloot'){flushRewards();const r=claimDungeonLoot(p);p.sendMessage(`§a던전 보급품 ${r.delivered}개 수령 · 보관 중 ${r.pending}개`);}
        else if(text)enterDungeon(p,text);
        else openDungeons(p);
      }catch(error){p.sendMessage('§c[던전] '+String(error.message??error));}
      return;
    }
    case 'runes': openRuneMenu(p); return;
    case 'jobs': openJobMenu(p); return;
    case 'skillguide': openSkillGuide(p); return;
    case 'craft': openCraftingGuide(p); return;
    case 'upgrade': openUpgradeMenu(p); return;
    case 'wallet': openWalletMenu(p); return;
    case 'equipment': openEquipmentMenu(p); return;
    case 'guide': openBeginnerGuide(p); return;
    case "rulebook": giveRulebook(p); return;
    case "ping": p.sendMessage("§a[RPG] v0.5.78 스크립트 정상. 명령어가 목록에 안 보이면 !rpg help 를 사용하세요."); return;
    case "menu": return openStatusTablet(p);
    case "skillbook": openChameleonSkillbook(p); return;
    case "learn": { const r=learnChameleonSkill(p,text);p.sendMessage(r.ok?r.message:`§c${r.message}`);return; }
    case "swap": { const r=swapChameleonWeapon(p);p.sendMessage(r.ok?r.message:`§c${r.message}`);return; }
    case "stats": openStatsMenu(p); return;
    case "admin": giveAdminTool(p); return;
    case "setlevel": {
      // /scriptevent 경로에서도 관리자 이외의 강제 설정을 막는다.
      if(!isAdminPlayer(p)){
        p.sendMessage('§c[RPG] 관리자만 강제 레벨을 설정할 수 있습니다.');return;
      }
      const result=forceSetLevel(p,text);
      if(!result.ok)p.sendMessage('§c[RPG] '+result.reason);
      else {
        refreshHud(p, true);
        p.sendMessage(`§a[RPG] 레벨 변경 완료: ${result.old} → ${result.level}${result.resetStats?' §e(초과 스탯 초기화)':''} §7(/rpg:stats 에서 확인)`);
      }
      return;
    }
    case "setmana": {
      if(!isAdminPlayer(p)){
        p.sendMessage('§c[RPG] 관리자만 강제 마나를 설정할 수 있습니다.');return;
      }
      const result=forceSetManaMax(p,text);
      if(!result.ok)p.sendMessage('§c[RPG] '+result.reason);
      else {
        refreshHud(p,true);
        p.sendMessage(`§b[RPG] ${result.reset?'마나 강제 설정 해제':'마나 설정 완료'}: 현재 ${result.current} / 최대 ${result.max} §7(/rpg:status 에서 확인)`);
      }
      return;
    }
    case "techgolem": {
      if(!isAdminPlayer(p)){p.sendMessage("§c[RPG] 관리자만 보스를 소환할 수 있습니다.");return;}
      try{
        const loc=p.location,view=p.getViewDirection(),flat=Math.hypot(view.x,view.z)||1;
        p.dimension.spawnEntity("rpg:tech_block_golem_boss",{x:loc.x+view.x/flat*11,y:loc.y+0.2,z:loc.z+view.z/flat*11});
        p.sendMessage("§b마공학 블럭 골렘이 소환되었습니다! §a떠 있는 블럭 관절로 이루어진 거대 보스입니다.");
      }catch(e){p.sendMessage("§c마공학 블럭 골렘 소환 실패: "+String(e));}
      return;
    }
    case "hydra": {
      if(!isAdminPlayer(p)){p.sendMessage("§c[RPG] 관리자만 보스를 소환할 수 있습니다.");return;}
      try{
        const loc=p.location,view=p.getViewDirection(),flat=Math.hypot(view.x,view.z)||1;
        p.dimension.spawnEntity("rpg:hydra_boss",{x:loc.x+view.x/flat*10,y:loc.y+0.2,z:loc.z+view.z/flat*10});
        p.sendMessage("§6지옥의 히드라가 소환되었습니다! §c세 개의 머리와 거대한 날개를 가진 필드 보스입니다.");
      }catch(e){p.sendMessage("§c히드라 소환 실패: "+String(e));}
      return;
    }
    case "cyclops": {
      if(!isAdminPlayer(p)){p.sendMessage("§c[RPG] 관리자만 보스를 소환할 수 있습니다.");return;}
      try{
        const loc=p.location,view=p.getViewDirection(),flat=Math.hypot(view.x,view.z)||1;
        p.dimension.spawnEntity("rpg:cyclops_boss",{x:loc.x+view.x/flat*9,y:loc.y+0.2,z:loc.z+view.z/flat*9});
        p.sendMessage("§6외눈박이 거인이 소환되었습니다! §c눈 약점에 명중하면 피해 3배");
      }catch(e){p.sendMessage("§c거인 소환 실패: "+String(e));}
      return;
    }
    case "boss": {
      try {
        const loc = p.location;
        const view = p.getViewDirection();
        const flat = Math.hypot(view.x,view.z)||1;
        p.dimension.spawnEntity("rpg:rock_golem_boss",{x:loc.x+view.x/flat*7,y:loc.y+0.2,z:loc.z+view.z/flat*7});
        p.sendMessage("§b바위 골렘이 소환되었습니다! §7처치 경험치 450 XP");
      }catch(e){p.sendMessage("§c골렘 소환 실패: "+String(e));}
      return;
    }
    case "guild":
      openGuildMenu(p); return;
    case "party":
      openPartyMenu(p); return;
    case "invites":
      openInvites(p); return;
    case "job":
    case "force": {
      const target = findJob(text);
      if (!target) {
        const names = Object.values(JOBS).map((j) => j.name).join(", ");
        p.sendMessage(`§c알 수 없는 직업입니다: ${text}\n§7가능: ${names}`);
        return;
      }
      requestAdvance(p, target, action === "force");
      return;
    }
    case "reset":
      resetToApprentice(p);
      p.sendMessage("§7견습으로 초기화했습니다.");
      return;
    case "starter": {
      const r = giveStarter(p, text);
      p.sendMessage(r.ok ? "§a견습 시작 아이템과 스킬을 받았습니다." : `§c${r.reason}`);
      return;
    }
    case "hito": return text?giveHitoEgg(p,text):openHitoEggMenu(p);
    case "egg": openEggMenu(p); return;
    case "items": {
      const n = giveAllItems(p);
      p.sendMessage(n > 0 ? `§a애드온 전용 아이템 ${n}개를 받았습니다.` : "§7이미 모두 가지고 있습니다.");
      return;
    }
    case "charge":
      setMana(p, maxManaOf(p));
      resetCooldowns(p);
      refreshHud(p, true);
      p.sendMessage("§b마나를 가득 채우고 쿨타임을 초기화했습니다.");
      return;
    case "offer":
      offerItems(p);
      return;
    case "bless":
      offerLight(p);
      return;
    case "holywater":
      craftHolyWater(p);
      return;
    case "wings":
      toggleWings(p);
      return;
    case "fistcheck":
      p.sendMessage(getFistCoatingStatus(p));
      return;
    case "wingcheck":
      inspectWings(p);
      return;
    case "wingrefresh":
      refreshWings(p);
      return;
    case "skills":
      p.sendMessage("§7채팅창을 닫으면 스킬 설정창이 열립니다.");
      openSkillMenu(p);
      return;
    case "status": {
      const job = JOBS[getJobId(p)];
      const skills = getSkills(p).map((id) => SKILLS[id]?.name ?? id).join(", ") || "없음";
      const off = getDisabled(p).map((id) => SKILLS[id]?.name ?? id).join(", ");
      const prog = getProgress(p);
      let msg =
        `§e직업: ${job?.name ?? "없음"} §7/ 마나 ${manaText(p)}/${maxManaText(p)}` +
        ` §7/ 책 ${prog.books ?? 0}권\n§7스킬: ${skills}${off ? `\n§7꺼 둔 스킬: ${off}` : ""}`;
      // 지금 직업에서 갈 수 있는 전직과 조건 진행도
      if(job?.id==='angel'){const healing=readAngelHealing(p);msg+=`\n§e축복의 빛: 아군 회복 ${Number(healing.remainder.toFixed(1))}/300 HP · 보관 ${healing.pending}개`;}
      for (const t of Object.values(JOBS)) {
        if (t.parent !== job?.id || t.placeholder || t.hidden) continue;
        const st = requirementStatus(p, t);
        msg += `\n§7→ ${t.name}: ${st ? `${st.label} ${st.have}/${st.need}` : "조건 없음"}`;
      }
      p.sendMessage(msg + "\n" + statsSummary(p));
      return;
    }
    default:
      p.sendMessage(HELP);
  }
}
```

## RPG_BP_v0578/scripts/core/statusTablet.js

```javascript
import {world,system,ItemStack} from '@minecraft/server';
import {ActionFormData} from '@minecraft/server-ui';
import {getJobId,getMana,maxManaOf} from './playerData.js';
import {getLevel,getXp,xpForNext,getStats,LEVEL_MAX} from './stats.js';
import {JOBS} from '../data/jobs.js';
import {showForm} from './skillMenu.js';

export const STATUS_TABLET='rpg:status_tablet';
export const STATUS_TITLE='§bSTATUS §8| §f상태창';
const GIVEN='rpg:status_tablet_given_v1';
const open=new Set(),lastUse=new Map();
let actionHandler,registered=false;
export const STATUS_ACTIONS=Object.freeze([
 {action:'dungeon',text:'§b던전 입장\n§8던전 목록'},
 {action:'party',text:'§b파티\n§8생성 · 초대'},
 {action:'runes',text:'§b룬 장착\n§8보관 · 관리'},
 {action:'skillguide',text:'§b스킬 효과\n§8효과 · MP · 쿨'},
 {action:'craft',text:'§b아이템 조합법\n§8재료 · 배치'},
 {action:'guild',text:'§b길드 확인\n§8생성 · 멤버'},
 {action:'stats',text:'§b스탯 배분\n§8능력치 · SP'},
 {action:'equipment',text:'§b장비 정보\n§8갑옷 · 효과'},
 {action:'upgrade',text:'§b장비 강화\n§8강화 관리'},
 {action:'jobs',text:'§b전직 안내\n§8직업 · 조건'},
 {action:'pvp',text:'§bPvP 결투\n§8플레이어 선택'},
 {action:'wallet',text:'§b지갑 · XP 상점\n§8포션 · 촉매 구매'},
 {action:'invites',text:'§b받은 초대\n§8파티 · 길드'},
 {action:'transfer',text:'§b강화 이전\n§8추출 · 적용'},
 {action:'tpa',text:'§b이동 요청 TPA\n§8이동 시 1,000골드'},
 {action:'tpaaccept',text:'§b받은 이동 요청\n§8수락 · 거절'},
 {action:'refresh',text:'§a새로고침\n§8내 정보 갱신'},
 {action:'close',text:'§7닫기\n§8모험으로 돌아가기'}
]);
export function resourceBar(value,max,color='§b'){
 const ratio=Number.isFinite(value)&&Number.isFinite(max)&&max>0?Math.max(0,Math.min(1,value/max)):0;
 const n=Math.round(ratio*10);return color+'■'.repeat(n)+'§8'+'■'.repeat(10-n)+'§r';
}
const number=n=>String(Math.round(Number.isFinite(n)?n:0));
export function statusBody(p){
 const h=p.getComponent('minecraft:health'),mp=getMana(p),maxMp=maxManaOf(p);
 const level=getLevel(p),xp=getXp(p),next=xpForNext(level),s=getStats(p),capped=level>=LEVEL_MAX;
 const name=String(p.name).replace(/§./g,'').slice(0,22);
 return `§f${name}\n§b${JOBS[getJobId(p)]?.name??'견습'} §fLv.${level}\n\n`+
  `§fHP ${number(h?.currentValue??0)} / ${number(h?.effectiveMax??20)}\n${resourceBar(h?.currentValue??0,h?.effectiveMax??20)}\n`+
  `§fMP ${number(mp)} / ${number(maxMp)}\n${resourceBar(mp,maxMp)}\n`+
  `§fEXP ${capped?'MAX':number(xp)+' / '+number(next)}\n${resourceBar(capped?1:xp,capped?1:next,'§d')}\n\n`+
  `§7근력 §f${s.str}  §7마력 §f${s.mag}\n§7체력 §f${s.vit}  §7지능 §f${s.int}`;
}
export async function openStatusTablet(p){
 if(!p?.isValid||open.has(p.id)||!actionHandler)return;
 open.add(p.id);
 let next;
 try{
  while(p.isValid){
   const form=new ActionFormData().title(STATUS_TITLE).body(statusBody(p));
   for(const entry of STATUS_ACTIONS)form.button(entry.text);
   const res=await showForm(form,p);
   if(!res||res.canceled||!Number.isInteger(res.selection)||res.selection<0||res.selection>=STATUS_ACTIONS.length)return;
   const selected=STATUS_ACTIONS[res.selection].action;
   if(selected==='refresh')continue;
   if(selected!=='close')next=selected;
   break;
  }
 }catch(error){try{p.sendMessage('§c[상태창] '+String(error.message??error));}catch{}}
 finally{open.delete(p.id);}
 // Defer the next form so Bedrock can close this form first.
 if(next&&p.isValid)system.run(()=>{try{const result=actionHandler(next,p);Promise.resolve(result).catch(error=>{try{p.sendMessage('§c[상태창] '+String(error));}catch{}});}catch(error){try{p.sendMessage('§c[상태창] '+String(error));}catch{}}});
}
export function giveStatusTablet(p){
 if(!p?.isValid||p.getDynamicProperty(GIVEN))return;
 const inv=p.getComponent('minecraft:inventory')?.container;if(!inv)return;
 for(let i=0;i<inv.size;i++)if(inv.getItem(i)?.typeId===STATUS_TABLET){p.setDynamicProperty(GIVEN,true);return;}
 const remainder=inv.addItem(new ItemStack(STATUS_TABLET,1));
 if(remainder){p.sendMessage('§e[상태창] 인벤토리를 비운 후 재접속하면 지급됩니다. /rpg:menu로도 열 수 있습니다.');return;}
 p.setDynamicProperty(GIVEN,true);
 p.sendMessage('§b[상태창] 상태창 아이템을 지급했습니다. 손에 들고 사용해 모험 메뉴를 여세요.');
}
function useTablet(p){
 if(!p?.isValid||open.has(p.id)||(lastUse.get(p.id)??-100)>system.currentTick-5)return;
 lastUse.set(p.id,system.currentTick);system.run(()=>openStatusTablet(p));
}
// Register the component in startup, before item definitions are used.
system.beforeEvents.startup.subscribe(event=>{
 try{event.itemComponentRegistry.registerCustomComponent('rpg:status_tablet',{onUse(e){useTablet(e.source);}});}
 catch(error){console.warn('[RPG status tablet component] '+error);}
});
export function registerStatusTablet(handler){
 actionHandler=handler;if(registered)return;registered=true;
 world.afterEvents.itemUse.subscribe(e=>{if(e.itemStack?.typeId===STATUS_TABLET)useTablet(e.source);});
 world.afterEvents.playerSpawn.subscribe(e=>system.runTimeout(()=>{try{giveStatusTablet(e.player);}catch(error){console.warn('[RPG status tablet grant] '+error);}},40));
 world.afterEvents.playerLeave.subscribe(e=>{open.delete(e.playerId);lastUse.delete(e.playerId);});
 system.runTimeout(()=>{for(const p of world.getAllPlayers())try{giveStatusTablet(p);}catch(error){console.warn('[RPG status tablet grant] '+error);}},40);
}
```

## RPG_BP_v0578/scripts/core/theWorldStand.js

```javascript
import {world,system} from '@minecraft/server';
import {getJobId} from './playerData.js';
const stands=new Map(),OWNER='rpg:the_world_owner';let registered=false;
export function tickTheWorldStands(){
 const players=world.getAllPlayers();
 for(const [id,e]of stands){try{if(!e.isValid){stands.delete(id);continue;}}catch{stands.delete(id);continue;}const p=players.find(p=>p.id===id);if(!p||getJobId(p)!=='the_world'||(p.getComponent('minecraft:health')?.currentValue??0)<=0||p.dimension.id!==e.dimension.id){try{e.remove();}catch{}stands.delete(id);}}
 for(const p of players){if(getJobId(p)!=='the_world'||(p.getComponent('minecraft:health')?.currentValue??0)<=0)continue;
  try{const rot=p.getRotation(),yaw=rot.y*Math.PI/180,at={x:p.location.x+Math.sin(yaw)*1.05,y:p.location.y+.15,z:p.location.z-Math.cos(yaw)*1.05};let e=stands.get(p.id);if(!e?.isValid){e=p.dimension.spawnEntity('rpg:the_world_stand',at);e.setDynamicProperty(OWNER,p.id);stands.set(p.id,e);}e.teleport(at,{dimension:p.dimension,rotation:{x:0,y:rot.y},keepVelocity:false});}catch(error){console.warn('[The World stand] '+error);}
 }
}
export function cleanWorldStands(){for(const name of ['overworld','nether','the_end'])try{for(const e of world.getDimension(name).getEntities({type:'rpg:the_world_stand'}))if(stands.get(e.getDynamicProperty(OWNER))?.id!==e.id)e.remove();}catch{}}
export function registerTheWorldStand(){if(registered)return;registered=true;system.run(cleanWorldStands);system.runInterval(tickTheWorldStands,2);system.runInterval(cleanWorldStands,100);world.afterEvents.playerLeave.subscribe(ev=>{try{stands.get(ev.playerId)?.remove();}catch{}stands.delete(ev.playerId);});}
```

## RPG_BP_v0578/scripts/core/timeStopDamage.js

```javascript
// No world imports: existing attacks remain unchanged unless this specific target is frozen by their source.
const records=new Map();
export function registerStoppedTarget(target,source){const record={target,source,damage:0};records.set(target.id,record);return record;}
export function deferStoppedDamage(target,source,amount){const r=records.get(target?.id);if(!r||r.source.id!==source?.id||!Number.isFinite(amount)||amount<=0)return false;r.damage+=amount;return true;}
export function takeStoppedDamage(targetId){const record=records.get(targetId);records.delete(targetId);return record;}
export function isTimeFrozen(entity){return records.has(entity?.id);}
```

## RPG_BP_v0578/scripts/core/tpa.js

```javascript
import {world,system} from '@minecraft/server';
import {ActionFormData,MessageFormData} from '@minecraft/server-ui';
import {showForm} from './skillMenu.js';
import {getWallet,setWallet} from './wallet.js';
import {duelOf} from './pvp.js';
import {dungeonRuns,dungeonLobbies} from './dungeonState.js';
import {isTimeFrozen} from './timeStopDamage.js';
export const TPA_PRICE=1000;const requests=new Map(),menus=new Set();let registered=false;
const online=id=>world.getAllPlayers().find(p=>p.id===id);
function unavailable(p){return !p?.isValid||(p.getComponent('minecraft:health')?.currentValue??0)<=0||duelOf(p)||isTimeFrozen(p)||[...dungeonRuns.values(),...dungeonLobbies.values()].some(r=>r.members.some(m=>m.id===p.id))||p.getDynamicProperty('rpg:pvp_arena_return_v1')!==undefined||p.getDynamicProperty('rpg:dungeon_return_v1')!==undefined;}
export function requestTpa(p,target){
 if(p.id===target.id||unavailable(p)||unavailable(target))throw Error('던전·결투·시간 정지·사망 중에는 이동 요청을 할 수 없습니다.');
 if(getWallet(p).gold<TPA_PRICE)throw Error('이동 시 사용할 1,000골드가 필요합니다.');
 requests.set(p.id,{from:p.id,to:target.id,until:Date.now()+30000});p.sendMessage('§b'+target.name+'님에게 이동을 요청했습니다. 수락·이동 성공 시 1,000골드를 사용합니다.');target.sendMessage('§e'+p.name+'님의 이동 요청입니다. 상태창의 받은 이동 요청 또는 /rpg:tpaaccept에서 30초 안에 수락하세요.');
}
export function safeTpaDestination(target){
 const base=target.location,dim=target.dimension;
 for(const dy of [0,1,-1,2,-2])for(const [dx,dz]of [[1,0],[-1,0],[0,1],[0,-1],[1,1],[-1,-1]]){
  const at={x:Math.floor(base.x)+dx+.5,y:Math.floor(base.y)+dy,z:Math.floor(base.z)+dz+.5};
  try{const floor=dim.getBlock({...at,y:at.y-1}),feet=dim.getBlock(at),head=dim.getBlock({...at,y:at.y+1});
   if(floor?.isSolid&&feet?.isAir&&head?.isAir&&!['minecraft:magma','minecraft:campfire','minecraft:soul_campfire','minecraft:cactus'].includes(floor.typeId))return at;
  }catch{}
 }
 throw Error('상대 주변에 안전한 이동 공간이 없습니다.');
}
export function acceptTpa(target,fromId){
 const request=requests.get(fromId),p=online(fromId);
 if(!request||request.to!==target.id||request.until<Date.now())throw Error('유효한 이동 요청이 없습니다.');
 if(unavailable(p)||unavailable(target))throw Error('참가자의 던전·결투·시간 정지·생존 상태를 확인하세요.');
 const before=getWallet(p);if(before.gold<TPA_PRICE)throw Error('요청자의 골드가 부족합니다.');
 const at=safeTpaDestination(target),origin={dimension:p.dimension,location:{...p.location},rotation:p.getRotation()};requests.delete(fromId);let moved=false;
 try{p.teleport(at,{dimension:target.dimension,keepVelocity:false});moved=true;setWallet(p,{...before,gold:before.gold-TPA_PRICE});}
 catch(error){try{setWallet(p,before);}catch{}if(moved)try{p.teleport(origin.location,{dimension:origin.dimension,rotation:origin.rotation,keepVelocity:false});}catch(rollback){console.error('[RPG TPA return] '+rollback);}throw error;}
 p.sendMessage('§a이동 완료 · 1,000골드를 사용했습니다.');target.sendMessage('§a'+p.name+'님이 이동했습니다.');return true;
}
export async function openTpa(p){
 if(menus.has(p.id))return;menus.add(p.id);
 try{const targets=world.getAllPlayers().filter(t=>t.id!==p.id);const f=new ActionFormData().title('플레이어 이동 요청 · TPA').body('상대가 수락하면 상대 근처로 이동합니다.\n이동 성공 시 요청자의 RPG 골드 1,000을 사용합니다.\n요청·거절·만료에는 비용이 들지 않습니다.');for(const t of targets)f.button(t.name);f.button('닫기');const r=await showForm(f,p);if(r&&!r.canceled&&Number.isInteger(r.selection)&&r.selection>=0&&r.selection<targets.length)requestTpa(p,targets[r.selection]);}
 catch(error){p.sendMessage('§e[TPA] '+String(error.message??error));}finally{menus.delete(p.id);}
}
export async function openTpaInbox(p){
 if(menus.has(p.id))return;menus.add(p.id);
 try{const pending=[...requests.values()].filter(r=>r.to===p.id&&r.until>Date.now()&&online(r.from));const f=new ActionFormData().title('받은 이동 요청').body(pending.length?'요청자를 선택해 수락하거나 거절하세요.':'받은 이동 요청이 없습니다.');for(const r of pending)f.button(online(r.from).name);f.button('닫기');const pick=await showForm(f,p);if(!pick||pick.canceled||!Number.isInteger(pick.selection)||pick.selection<0||pick.selection>=pending.length)return;const req=pending[pick.selection];const reply=await showForm(new MessageFormData().title('이동 요청 수락').body(online(req.from).name+'님을 내 주변으로 이동시킬까요? 상대방이 1,000골드를 사용합니다.').button1('거절').button2('수락'),p);if(reply&&!reply.canceled){if(reply.selection===1)acceptTpa(p,req.from);else requests.delete(req.from);}}
 catch(error){p.sendMessage('§e[TPA] '+String(error.message??error));}finally{menus.delete(p.id);}
}
export function registerTpa(){if(registered)return;registered=true;world.afterEvents.playerLeave.subscribe(e=>{menus.delete(e.playerId);requests.delete(e.playerId);for(const [id,r]of requests)if(r.to===e.playerId)requests.delete(id);});system.runInterval(()=>{for(const [id,r]of requests)if(r.until<Date.now())requests.delete(id);},100);}
```

## RPG_BP_v0578/scripts/core/util.js

```javascript
import {deferStoppedDamage} from './timeStopDamage.js';
import {setCombatHealth} from './combatContext.js';
import {applyStatus,ccScale} from './crowdControl.js';
import {monsterDefenseMultiplier} from '../data/monsterLevels.js';
import {pvpBlocked} from './pvp.js';
import {scriptedHit,noteManualDamage,rememberScriptKiller} from './combatContext.js';
import { EntityDamageCause, EquipmentSlot } from "@minecraft/server";
import { hasPiercing } from "./piercing.js";
import { CONFIG } from "../config.js";
import { areAllies } from "./social.js";
import { skillDamageFor, defaultDamageTypeFor } from "./stats.js";

// 아이템/경험치 구슬 같은 것은 대상에서 제외합니다.
const EXCLUDE_TYPES = ["rpg:dungeon_loot_cache", "rpg:gpt_guide","minecraft:item", "minecraft:xp_orb", "minecraft:armor_stand", "rpg:moon_chakram_projectile", "rpg:moon_weapon_chakram", "rpg:tech_prison_dome"];

export function healthAlive(entity) {
  try {
    const hp = entity.getComponent("minecraft:health");
    return !!hp && hp.currentValue > 0;
  } catch {
    return false;
  }
}

// center 주변 radius 블록 안의 공격 가능한 엔티티 목록.
// 엔티티 위치는 발밑 기준이라, 눈높이로 날아가는 투사체처럼 center 가 몸 높이일 때는
// bodyOffset(보통 0.9)을 줘서 몸 중심 기준으로 거리를 잽니다.
export function getTargets(source, center, radius, bodyOffset = 0) {
  const out = [];
  let list = [];
  try {
    list = source.dimension.getEntities({
      location: { x: center.x, y: center.y - bodyOffset, z: center.z },
      maxDistance: radius,
      excludeTypes: EXCLUDE_TYPES,
    });
  } catch {
    return out;
  }
  for (const e of list) {
    try {
      if (e.typeId === "rpg:gpt_guide" || e.id === source.id || areAllies(source, e)) continue;
      if (!CONFIG.pvp && e.typeId === "minecraft:player") continue;
      if (!healthAlive(e)) continue;
      out.push(e);
    } catch {
      // 사라진 엔티티는 무시
    }
  }
  return out;
}

export function hurt(target,source,amount,type="auto",options){return scriptedHit(target,source,()=>hurtCore(target,source,amount,type,options));}
function hurtCore(target, source, amount, type = "auto", options) {
  if(pvpBlocked(source,target))return false;
  if (target?.typeId==="rpg:gpt_guide"||areAllies(source, target)) return false;
  if (!CONFIG.pvp && target?.typeId === "minecraft:player") return false;
  const finalDamage=skillDamageFor(source, amount, type === "auto" ? defaultDamageTypeFor(source) : type, options);
  if(deferStoppedDamage(target,source,finalDamage))return true;
  const defense=monsterDefenseMultiplier(target),engineDamage=type==='fixed'?finalDamage/defense:finalDamage;
  const directDamage=type==='fixed'?finalDamage:finalDamage*defense;
  if(hasPiercing(source)||options?.ignoreInvulnerability){
    // override is used for 3 seconds only. Fallback handles damage immunity
    // when an engine still rejects repeated hits via applyDamage.
    try{
      if(target.applyDamage(engineDamage,{cause:EntityDamageCause.override,damagingEntity:source})) return true;
    }catch{}
    try{
      const hp=target.getComponent("minecraft:health");
      if(hp && hp.currentValue>0){
        const before=hp.currentValue;if(directDamage>=before)rememberScriptKiller(source,target);setCombatHealth(target,source,hp,Math.max(0,before-directDamage));noteManualDamage(source,target,Math.min(before,directDamage));
        return true;
      }
    }catch{}
    return false;
  }
  try {
    return target.applyDamage(engineDamage,{
      cause: EntityDamageCause.entityAttack,
      damagingEntity: source,
    }) !== false;
  } catch {
    return false;
  }
}

// 버전에 따라 applyKnockback 인자 형태가 달라서 두 가지를 모두 시도합니다.
export function knockback(entity, dirX, dirZ, strength, vertical = 0.35) {
  strength*=ccScale(entity);vertical*=ccScale(entity);
  const len = Math.hypot(dirX, dirZ) || 1;
  const x = dirX / len;
  const z = dirZ / len;
  try {
    entity.applyKnockback({ x: x * strength, z: z * strength }, vertical);
    return;
  } catch {
    // 구버전 형태로 재시도
  }
  try {
    entity.applyKnockback(x, z, strength, vertical);
  } catch {
    // 밀 수 없는 대상
  }
}

// 엔티티에 불 붙이기. 블록/바닥에는 불이 붙지 않습니다.
export function burn(entity, seconds) {
  if (!CONFIG.igniteEntities) return;
  try {
    entity.setOnFire(seconds, true);
  } catch {
    // 불 붙일 수 없는 대상
  }
}

export function slow(entity, amplifier, ticks) {
  try {
    applyStatus(entity,"slowness", ticks, { amplifier, showParticles: true });
  } catch {
    // 효과를 받을 수 없는 대상
  }
}

export function push(entity, x, y, z) {
  const scale=ccScale(entity);x*=scale;y*=scale;z*=scale;
  try {
    entity.applyImpulse({ x, y, z });
  } catch {
    // 플레이어 등 impulse 불가 대상
  }
}

export function particle(dimension, id, pos) {
  try {
    dimension.spawnParticle(id, pos);
  } catch {
    // 없는 파티클 id 등은 무시
  }
}

export function sound(dimension, id, pos, options = { volume: 1, pitch: 1 }) {
  try {
    dimension.playSound(id, pos, options);
  } catch {
    // 없는 사운드 id 등은 무시
  }
}

// 시선의 수평 방향 (위/아래를 봐도 앞쪽으로 나가도록)
export function flatDir(player) {
  const v = player.getViewDirection();
  const len = Math.hypot(v.x, v.z);
  if (len < 0.001) {
    const yaw = (player.getRotation().y * Math.PI) / 180;
    return { x: -Math.sin(yaw), z: Math.cos(yaw) };
  }
  return { x: v.x / len, z: v.z / len };
}

export function isSolidAt(dimension, pos) {
  try {
    const b = dimension.getBlock(pos);
    if (!b) return true;
    const solid = typeof b.isSolid === "function" ? b.isSolid() : b.isSolid;
    return !!solid;
  } catch {
    return true;
  }
}

export function dist3(a, b) {
  return Math.hypot(a.x - b.x, a.y - b.y, a.z - b.z);
}

export function norm3(v) {
  const l = Math.hypot(v.x, v.y, v.z) || 1;
  return { x: v.x / l, y: v.y / l, z: v.z / l };
}

export function headPos(player) {
  try {
    return player.getHeadLocation();
  } catch {
    const l = player.location;
    return { x: l.x, y: l.y + 1.6, z: l.z };
  }
}

// 시선 끝 지점 (벽/바닥에 닿으면 그 직전). hit 는 무언가에 닿았는지.
export function aimPoint(player, maxDist, step = 0.5) {
  const dim = player.dimension;
  const d = player.getViewDirection();
  const h = headPos(player);
  let last = { ...h };
  for (let t = step; t <= maxDist; t += step) {
    const p = { x: h.x + d.x * t, y: h.y + d.y * t, z: h.z + d.z * t };
    if (isSolidAt(dim, p)) return { point: last, hit: true };
    last = p;
  }
  return { point: last, hit: false };
}

// 시선 앞에서 가장 먼저 만나는 대상
export function firstTargetInView(player, maxDist) {
  const dim = player.dimension;
  const d = player.getViewDirection();
  const h = headPos(player);
  for (let t = 1; t <= maxDist; t += 1) {
    const p = { x: h.x + d.x * t, y: h.y + d.y * t, z: h.z + d.z * t };
    if (isSolidAt(dim, p)) return undefined;
    const list = getTargets(player, p, 2.2, 0.9);
    if (list.length > 0) {
      list.sort((a, b) => dist3(a.location, h) - dist3(b.location, h));
      return list[0];
    }
  }
  return undefined;
}

// 현재 주 손에 든 아이템의 typeId
export function heldItemId(player) {
  try {
    const eq = player.getComponent("minecraft:equippable");
    const item = eq?.getEquipment(EquipmentSlot.Mainhand ?? "Mainhand");
    return item?.typeId;
  } catch {
    return undefined;
  }
}
```

## RPG_BP_v0578/scripts/data/classWeapons.js

```javascript
// Class items have enhancement only; deliberately absent from itemRarity.
export const CLASS_WEAPONS=Object.freeze([
  {id:'rpg:the_world_arrow',name:'더 월드의 화살 · 전용 건틀릿',damage:0},
  {
    "id": "rpg:apprentice_sword",
    "name": "견습의 검",
    "damage": 8
  },
  {
    "id": "rpg:apprentice_book",
    "name": "견습의 마법책",
    "damage": 0
  },
  {
    "id": "rpg:apprentice_cross",
    "name": "견습의 십자가",
    "damage": 0
  },
  {
    "id": "rpg:apprentice_fist",
    "name": "견습의 주먹",
    "damage": 7
  },
  {
    "id": "rpg:apprentice_bow",
    "name": "견습 활",
    "damage": 0
  },
  {
    "id": "rpg:war_god_gauntlets",
    "name": "투신의 흑염 건틀릿",
    "damage": 18
  },
  {
    "id": "rpg:swordsman_sword",
    "name": "검사의 검",
    "damage": 10
  },
  {
    "id": "rpg:sword_saint_sword",
    "name": "검성의 검",
    "damage": 13
  },
  {
    "id": "rpg:fallen_sword_saint_sword",
    "name": "타락검성의 검",
    "damage": 9
  },
  {
    "id": "rpg:mage_book",
    "name": "마법사의 마법서",
    "damage": 0
  },
  {
    "id": "rpg:sage_staff",
    "name": "현자의 지팡이",
    "damage": 0
  },
  {
    "id": "rpg:archer_bow",
    "name": "궁수 활",
    "damage": 0
  },
  {
    "id": "rpg:heavenly_bow",
    "name": "천궁 활",
    "damage": 0
  },
  {
    "id": "rpg:mangnani_axe",
    "name": "망나니의 전투 도끼",
    "damage": 9
  },
  {
    "id": "rpg:beast_axe",
    "name": "야수의 전투 도끼",
    "damage": 12
  },
  {
    "id": "rpg:frost_gauntlet",
    "name": "냉기 능력자의 서리 손",
    "damage": 7
  },
  {
    "id": "rpg:fire_gauntlet",
    "name": "화염술사의 불꽃 손",
    "damage": 7
  },
  {
    "id": "rpg:dragon_knight_lance",
    "name": "용기사의 창",
    "damage": 10
  },
  {
    "id": "rpg:demon_scythe",
    "name": "악마의 영혼 낫",
    "damage": 11
  },
  {
    "id": "rpg:tech_staff",
    "name": "마법공학사의 지팡이",
    "damage": 6
  },
  {
    "id": "rpg:hacker_core",
    "name": "해커 코어",
    "damage": 4
  },
  {
    "id": "rpg:blood_crystal",
    "name": "혈술사의 피 결정",
    "damage": 5
  },
  {
    "id": "rpg:assassin_blade",
    "name": "암살자의 단검",
    "damage": 8
  },
  {
    "id": "rpg:shadow_assassin_blade",
    "name": "그림자 암살자의 단검",
    "damage": 10
  },
  {
    "id": "rpg:harpooner_harpoon",
    "name": "작살꾼의 작살",
    "damage": 7
  },
  {
    "id": "rpg:revenant_harpoon",
    "name": "복수귀의 영혼작살",
    "damage": 11
  },
  {
    "id": "rpg:sun_heir_axe",
    "name": "해의 한손 도끼",
    "damage": 12
  },
  {
    "id": "rpg:moon_heir_blade",
    "name": "달의 차크람",
    "damage": 11
  },
  {
    "id": "rpg:chameleon_flame_blade",
    "name": "흑염 대검",
    "damage": 12
  },
  {
    "id": "rpg:fighter_fist",
    "name": "무투가의 주먹",
    "damage": 10
  },
  {
    "id": "rpg:fist_bone_fist",
    "name": "권골의 주먹",
    "damage": 13
  },
  {
    "id": "rpg:cleric_cross",
    "name": "성직자의 성십자",
    "damage": 0
  },
  {
    "id": "rpg:angel_hands",
    "name": "천사의 기도",
    "damage": 0
  },
  {
    "id": "rpg:michael_wings",
    "name": "미카엘의 날개",
    "damage": 0
  },
  {
    "id": "rpg:fallen_angel_wings",
    "name": "타락천사의 날개",
    "damage": 0
  },
  {
    "id": "rpg:chameleon_moon_scythe",
    "name": "백월의 낫",
    "damage": 12
  }
]);
export const classWeapon=id=>CLASS_WEAPONS.find(w=>w.id===id);
```

## RPG_BP_v0578/scripts/data/hudNames.js

```javascript
export const HUD_ITEM_NAMES={
 'rpg:the_world_arrow':'더 월드의 화살',"rpg:war_god_gauntlets":"투신의 흑염 건틀릿","rpg:mangnani_axe": "망나니의 전투 도끼", "rpg:warden_leggings": "숲 수호자 각반", "rpg:blessing_light": "축복의 빛", "rpg:dragon_knight_lance": "용기사의 창", "rpg:fallen_sword_saint_sword": "타락검성의 검", "rpg:warden_boots": "숲 수호자 장화", "rpg:shadow_assassin_blade": "그림자 암살자의 단검", "rpg:hacker_core": "해커 코어", "rpg:sandbag_spawner": "훈련용 샌드백 소환", "rpg:adventurer_leggings": "모험가 각반", "rpg:admin_settings": "§6RPG 운영자 관리 도구", "rpg:holy_water": "성수", "rpg:archer_bow": "궁수 활", "rpg:rune_ward": "수호의 룬", "rpg:angel_wing_toggle": "천사 날개", "rpg:fighter_fist": "무투가의 주먹", "rpg:blood_crystal": "혈술사의 피 결정", "rpg:mage_book": "마법사의 마법서", "rpg:random_skillbook": "카멜레온 랜덤 스킬북", "rpg:astral_boots": "별빛 술사 장화", "rpg:adventurer_chestplate": "모험가 흉갑", "rpg:fallen_angel_wings": "타락천사의 날개", "rpg:astral_leggings": "별빛 술사 각반", "rpg:blood_spear": "피의 창", "rpg:michael_wings": "미카엘의 날개", "rpg:frost_gauntlet": "냉기 능력자의 서리 손", "rpg:sun_heir_axe": "해의 한손 도끼", "rpg:chameleon_moon_scythe": "백월의 낫", "rpg:adventurer_helmet": "모험가 투구", "rpg:adventurer_boots": "모험가 장화", "rpg:harpooner_harpoon": "작살꾼의 작살", "rpg:sage_staff": "현자의 지팡이", "rpg:apprentice_sword": "견습의 검", "rpg:apprentice_fist": "견습의 주먹", "rpg:swordsman_sword": "검사의 검", "rpg:moon_heir_focus": "달빛 인장 (스킬 사용)", "rpg:rune_power": "힘의 룬", "rpg:fallen_wing_toggle": "타락한 날개", "rpg:chameleon_flame_blade": "흑염 대검", "rpg:rune_haste": "이전 돌풍 룬 (샘의 룬으로 전환)", "rpg:astral_helmet": "별빛 술사 투구", "rpg:deathlord_staff": "반물질의 신 지팡이", "rpg:mangnani_contract": "망나니 전직서 (해금 미설정)", "rpg:assassin_blade": "암살자의 단검", "rpg:angel_hands": "천사의 기도", "rpg:enhancement_catalyst": "강화 촉매", "rpg:moon_heir_blade": "달의 차크람", "rpg:beast_axe": "야수의 전투 도끼", "rpg:fire_gauntlet": "화염술사의 불꽃 손", "rpg:rune_arcane": "비전의 룬", "rpg:sword_saint_sword": "검성의 검", "rpg:fist_bone_fist": "권골의 주먹", "rpg:apprentice_cross": "견습의 십자가", "rpg:apprentice_bow": "견습 활", "rpg:heavenly_bow": "천궁 활", "rpg:necromancer_contract": "흑마법사 전직서 (해금 미설정)", "rpg:rune_vitality": "생명의 룬", "rpg:rune_mana": "샘의 룬", "rpg:revenant_harpoon": "복수귀의 영혼작살", "rpg:apprentice_tome": "견습 사령서", "rpg:necromancer_staff": "흑마법사 지팡이", "rpg:demon_scythe": "악마의 영혼 낫", "rpg:apprentice_book": "견습의 마법책", "rpg:cleric_cross": "성직자의 성십자", "rpg:michael_wing_toggle": "미카엘의 날개", "rpg:warden_helmet": "숲 수호자 투구", "rpg:tech_aux_arm": "마법공학 보조 기계팔", "rpg:warden_chestplate": "숲 수호자 흉갑", "rpg:astral_chestplate": "별빛 술사 흉갑", "rpg:tech_staff": "마법공학사의 지팡이"};
```

## RPG_BP_v0578/scripts/data/jobs.js

```javascript
// v0.5.78 실제 전직 시련은 data/jobTrials.js를 사용합니다. req는 이전 저장값과 일부 명중 안내용 메타데이터입니다.
// 직업 트리. 새 직업/4차 전직은 여기에 항목만 추가하면 됩니다.
//   tier     : 전직 차수 (1=견습, 2, 3 ...)
//   line     : 계열. 2차 전직 시 이 계열 밖의 스킬은 사라집니다.
//   parent   : 이 직업으로 전직하기 직전의 직업
//   weapon   : 전직 시 1번 칸에 고정되는 무기 아이템 (없으면 null)
//   skills   : 이 직업이 쓰는 스킬 id 목록 (웅크리기로 순서대로 전환)
//   basic    : 휘두르기로 푸른 에너지를 쏘는지
//   corrupt  : 타락 직업 여부
//   highMana : 마법사/성직자 계열 (마나 상한 2000, 책 1권당 +20)
//   theme    : 이펙트 색 (blue / red)
//   req      : 전직 조건 { key(진행도 이름), need(필요 수), label(설명) }
//   placeholder : true 면 아직 구현 전인 직업 (전직 불가)

// 견습이 계열별로 받는 시작 아이템. 든 아이템에 따라 쓸 수 있는 스킬 목록이 달라집니다.
export const STARTER_ITEMS = {
  "rpg:apprentice_sword": "sword",
  "rpg:apprentice_book": "mage",
  "rpg:apprentice_cross": "holy",
  "rpg:apprentice_fist": "fist",
  "rpg:apprentice_bow": "archer",
};
export const STARTER_BY_LINE = {
  sword: "rpg:apprentice_sword",
  mage: "rpg:apprentice_book",
  holy: "rpg:apprentice_cross",
  fist: "rpg:apprentice_fist",
  archer: "rpg:apprentice_bow",
};
// 견습이 시작 아이템과 함께 배우는 스킬
export const APPRENTICE_SKILLS = {
  sword: [],
  mage: ["mage_fireball", "mage_lightning"],
  holy: ["holy_heal"],
  fist: [],
  archer: ["archer_airwalk", "archer_rapid"],
};

const SWORD_SAINT_SKILLS = ["sword_spin", "sword_wave", "sword_flame_spin", "sword_kenho_rando", "sword_moon_dragon", "sword_annihilation", "sword_rush"];
// 타락검성은 섬멸을 사용할 수 없고 광분화를 배웁니다.
const FALLEN_SWORD_SKILLS = ["sword_spin", "sword_wave", "sword_flame_spin", "sword_kenho_rando", "sword_moon_dragon", "sword_rush", "fallen_sword_frenzy"];

const MAGE_2 = ["mage_flame_on", "mage_thunder", "mage_ice_fire", "mage_gravity", "mage_bombaye"];
const SAGE_3 = [
  "sage_meteor", "sage_raiyoung", "sage_zero", "sage_blackhole", "sage_sphere", "sage_zoltraak",
];

// 성직자 계열
const CLERIC_SKILLS = ["holy_heal", "holy_shield", "holy_sword", "holy_orb", "holy_pray"];
const ANGEL_SKILLS = [...CLERIC_SKILLS, "angel_bless", "angel_zone"];
const FALLEN_SKILLS = [
  "fallen_sacrifice", "fallen_sword", "fallen_shield", "fallen_curse",
  "fallen_orb", "fallen_zone", "fallen_end",
];

// 무투가 계열 (견습 → 무투가 → 권골)
const FIGHTER_SKILLS = ["fist_strong", "fist_flurry", "fist_smash"];
const FIST_BONE_SKILLS = [
  ...FIGHTER_SKILLS, "fist_wind", "fist_face", "fist_stance", "fist_finale",
];

function stub(id, name, tier, line, parent, extra = {}) {
  return {
    id, name, tier, line, parent,
    weapon: null, skills: [], basic: false, corrupt: false, highMana: false, theme: "blue",
    placeholder: true,
    ...extra,
  };
}

export const JOBS = {
  the_world:{id:'the_world',name:'더 월드',tier:2,line:'the_world',parent:null,hidden:true,unlockPending:true,weapon:'rpg:the_world_arrow',skills:['world_muda','world_finish','world_roller','world_high','world_stop'],basic:false,corrupt:false,highMana:false,theme:'red'},
  war_god:{id:'war_god',name:'투신',tier:2,line:'war_god',parent:null,hidden:true,weapon:'rpg:war_god_gauntlets',skills:['war_god_punch','war_god_flurry','war_god_kick','war_god_kick_flurry','war_god_black_flash'],basic:false,corrupt:false,highMana:false,theme:'red'},
  apprentice: {
    id: "apprentice", name: "견습", tier: 1, line: "all", parent: null,
    weapon: null, skills: [], basic: false, corrupt: false, highMana: false, theme: "blue",
  },

  // ---- 검 계열 ----
  swordsman: {
    id: "swordsman", name: "검사", tier: 2, line: "sword", parent: "apprentice",
    weapon: "rpg:swordsman_sword", skills: ["sword_spin", "sword_wave", "sword_flame_spin"], basic: false, corrupt: false, highMana: false, theme: "blue",
    req: { key: "sword_kills", need: 100, label: "검으로 엔티티 처치" },
  },
  sword_saint: {
    id: "sword_saint", name: "검성", tier: 3, line: "sword", parent: "swordsman",
    weapon: "rpg:sword_saint_sword", skills: SWORD_SAINT_SKILLS, basic: true, corrupt: false, highMana: false, theme: "blue",
    req: { key: "diamond_swords", need: 100, label: "다이아몬드 검 바치기 (/rpg:offer)" },
  },
  // 타락검성은 검성의 스킬과 전용 액티브 '광분화'를 사용합니다.
  fallen_sword_saint: {
    id: "fallen_sword_saint", name: "타락검성", tier: 3, line: "sword", parent: "sword_saint",
    weapon: "rpg:fallen_sword_saint_sword", skills: FALLEN_SWORD_SKILLS, basic: true, corrupt: true, highMana: false, theme: "red",
    req: { key: "villager_kills", need: 100, label: "검성 상태에서 주민 처치" },
  },

  // ---- 마법 계열 ----
  mage: {
    id: "mage", name: "마법사", tier: 2, line: "mage", parent: "apprentice",
    weapon: "rpg:mage_book", skills: ["mage_fireball", "mage_lightning", ...MAGE_2],
    basic: false, corrupt: false, highMana: true, theme: "blue",
    req: { key: "books", need: 64, label: "인챈트 책 바치기 (/rpg:offer)" },
  },
  sage: {
    id: "sage", name: "현자", tier: 3, line: "mage", parent: "mage",
    weapon: "rpg:sage_staff", skills: [...MAGE_2, ...SAGE_3],
    basic: false, corrupt: false, highMana: true, theme: "blue",
    req: { key: "mana_spent", need: 30000, label: "마법사 상태에서 마나 사용" },
  },

  // ---- 궁수 계열: 견습 궁수 → 궁수 → 천궁 ----
  archer: {
    id: "archer", name: "궁수", tier: 2, line: "archer", parent: "apprentice",
    weapon: "rpg:archer_bow", skills: ["archer_airwalk", "archer_rapid", "archer_fire", "archer_ice", "archer_lightning", "archer_pierce"],
    basic: false, corrupt: false, highMana: false, theme: "blue",
    req: { key: "apprentice_arrow_hits", need: 20, label: "견습 활 명중" },
  },
  heavenly_archer: {
    id: "heavenly_archer", name: "천궁", tier: 3, line: "archer", parent: "archer",
    weapon: "rpg:heavenly_bow", skills: ["archer_airwalk", "archer_rapid", "archer_ice", "archer_lightning", "archer_pierce", "archer_homing", "archer_bomb", "archer_judgement", "archer_heavenly"],
    basic: false, corrupt: false, highMana: false, theme: "blue",
    req: { key: "archer_arrow_hits", need: 80, label: "궁수 활 명중" },
  },


  // ---- 히든 망나니 -> 야수 (망나니 스킬 승계) ----
  mangnani: {
    id: "mangnani", name: "망나니", tier: 2, line: "berserker", parent: null,
    hidden: true, weapon: "rpg:mangnani_axe", skills: ["berserker_charge", "berserker_slam", "berserker_execute", "beast_berserk", "berserker_triple_slash"],
    passive: "berserker_kill",
    basic: false, corrupt: true, highMana: false, theme: "red",
  },
  beast: {
    id: "beast", name: "야수", tier: 3, line: "berserker", parent: "mangnani",
    weapon: "rpg:beast_axe", skills: ["berserker_charge", "berserker_slam", "berserker_execute", "beast_berserk", "berserker_triple_slash", "beast_flame_burst", "beast_grand_explosion"],
    passive: "berserker_kill",
    basic: false, corrupt: true, highMana: false, theme: "red",
  },

  // ---- 히든 직업 냉기 능력자 (해금 방법은 사용자가 나중에 지정) ----
  frost_controller: {
    id: "frost_controller", name: "냉기 능력자", tier: 2, line: "frost", parent: null,
    hidden: true, weapon: "rpg:frost_gauntlet",
    skills: ["frost_shard", "frost_grasp", "frost_barrage", "frost_step", "frost_field", "frost_great_release", "frost_absolute"],
    basic: false, corrupt: false, highMana: true, theme: "blue",
  },

  // ---- 히든 직업 화염술사 (해금 조건 추후 결정) ----
  fire_controller: {
    id: "fire_controller", name: "화염술사", tier: 2, line: "fire", parent: null,
    hidden: true, weapon: "rpg:fire_gauntlet",
    skills: ["fire_ignite", "fire_pillar", "fire_wheel", "fire_armor", "fire_emperor"],
    basic: false, corrupt: false, highMana: true, theme: "red",
  },


  // ---- 히든 직업 용기사 ----
  dragon_knight: {
    id: "dragon_knight", name: "용기사", tier: 2, line: "dragon", parent: null,
    hidden: true, weapon: "rpg:dragon_knight_lance",
    skills: ["dragon_triple_strike", "dragon_gaze", "dragon_charge", "dragon_sprint", "dragon_breath", "dragon_domain"],
    passive: "dragon_third_strike",
    basic: false, corrupt: false, highMana: false, theme: "blue",
  },

  // ---- 악마: 검붉은 낫과 영혼 수확 패시브 ----
  demon: {
    id: "demon", name: "악마", tier: 2, line: "demon", parent: null, hidden: true,
    weapon: "rpg:demon_scythe",
    skills: ["demon_fear", "demon_vampirism", "demon_reap", "demon_contract", "demon_storm", "demon_domain"],
    passive: "demon_soul_harvest", basic: false, corrupt: true, highMana: true, theme: "red",
  },

  // ---- 마법공학사: 등 뒤의 보조 기계팔 + 표식 기반 기계 마법 ----
  technomancer: {
    id: "technomancer", name: "마법공학사", tier: 2, line: "tech", parent: null, hidden: true,
    weapon: "rpg:tech_staff", skills: ["tech_ray","tech_prison","tech_sphere","tech_bomb","tech_storm"],
    basic: false, corrupt: false, highMana: true, theme: "blue",
  },

  // ---- 신규 직업: 해커 / 혈술사 / 암살자 -> 그림자 암살자 ----
  // 해금 조건과 전용 스킬은 사용자 설정을 위해 현재 미지정.
  hacker: {
    id: "hacker", name: "해커", tier: 2, line: "hacker", parent: null, hidden: true,
    weapon: "rpg:hacker_core", skills: ["hacker_hack", "hacker_dummy_data", "hacker_reboot", "hacker_stealth", "hacker_copy", "hacker_ddos", "hacker_emp"], basic: false, corrupt: false, highMana: true, theme: "blue",
  },
  blood_mage: {
    id: "blood_mage", name: "혈술사", tier: 2, line: "blood", parent: null, hidden: true,
    weapon: "rpg:blood_crystal", skills: ["blood_spear", "blood_orb", "blood_supernova", "blood_mark", "blood_awaken", "blood_bind"], basic: false, corrupt: false, highMana: true, theme: "red",
  },
  assassin: {
    id: "assassin", name: "암살자", tier: 2, line: "assassin", parent: null, hidden: true,
    weapon: "rpg:assassin_blade", skills: ["assassin_spin", "assassin_dagger_wave", "assassin_pierce"], basic: false, corrupt: false, highMana: false, theme: "blue",
  },
  shadow_assassin: {
    id: "shadow_assassin", name: "그림자 암살자", tier: 3, line: "assassin", parent: "assassin",
    weapon: "rpg:shadow_assassin_blade", skills: ["assassin_spin", "assassin_dagger_wave", "assassin_pierce", "assassin_stealth", "assassin_mark", "shadow_assassinate"], basic: false, corrupt: false, highMana: false, theme: "red",
  },

  // ---- 작살 계열: 작살꾼 → 복수귀 ----
  harpooner: {
    id: "harpooner", name: "작살꾼", tier: 2, line: "harpoon", parent: null, hidden: true,
    weapon: "rpg:harpooner_harpoon", skills: ["harpoon_throw", "harpoon_dive", "harpoon_charge"],
    basic: false, corrupt: false, highMana: false, theme: "blue",
  },
  revenant: {
    id: "revenant", name: "복수귀", tier: 3, line: "harpoon", parent: "harpooner",
    weapon: "rpg:revenant_harpoon", skills: ["revenant_deep_pierce", "revenant_rend", "harpoon_throw", "harpoon_charge", "soul_harpoon", "harpoon_dive", "soul_separation", "revenant_execution"],
    basic: false, corrupt: true, highMana: false, theme: "blue",
  },

  // ---- 해의 후예 / 달의 후예: 지상의 햇빛·달빛 충전 계열 ----
  sun_heir: {
    id:"sun_heir", name:"해의 후예", tier:2, line:"celestial_sun", parent:null,
    weapon:"rpg:sun_heir_axe", highMana:true, hidden:true, basic:false, corrupt:false, theme:"red",
    skills:["sun_charge","sun_slash","sun_blessing","sun_divine_blessing","sun_meteor_drive","sun_radiant_beam","sun_descent"],
  },
  moon_heir: {
    id:"moon_heir", name:"달의 후예", tier:2, line:"celestial_moon", parent:null,
    weapon:"rpg:moon_heir_blade", highMana:true, hidden:true, basic:false, corrupt:false, theme:"blue",
    skills:["moon_charge","moon_slash","moon_moonlight","moon_homing","moon_chakram_split","moon_veil","moon_fullwave"],
  },

  // ---- 카멜레온: 두 무기 전환 / 다른 직업의 모든 액티브 스킬 학습 ----
  chameleon: {
    id: "chameleon", name: "카멜레온", tier: 2, line: "chameleon", parent: null, hidden: true,
    weapon: "rpg:chameleon_flame_blade", altWeapon: "rpg:chameleon_moon_scythe",
    skills: ["chameleon_crimson_dash", "chameleon_ground_slam"], basic: false, corrupt: false, highMana: true, theme: "red",
  },

  // ---- 환수종 (미구현 자리 표시자, 전직 불가) ----
  // 삭제한 반인반수와의 전직 연결은 끊었습니다.
  beast_lord: stub("beast_lord", "환수종", 3, "beast", null),

  // ---- 무투가 계열 ----  (두 직업 모두 낙하 피해 없음)
  fighter: {
    id: "fighter", name: "무투가", tier: 2, line: "fist", parent: "apprentice",
    weapon: "rpg:fighter_fist", skills: FIGHTER_SKILLS, basic: false, corrupt: false, highMana: false, theme: "red",
    req: { key: "apprentice_sandbags", need: 1, label: "견습 상태에서 샌드백 파괴" },
  },
  fist_bone: {
    id: "fist_bone", name: "권골", tier: 3, line: "fist", parent: "fighter",
    weapon: "rpg:fist_bone_fist", skills: FIST_BONE_SKILLS, basic: false, corrupt: false, highMana: false, theme: "red",
    req: { key: "fighter_sandbags", need: 10, label: "무투가 상태에서 샌드백 10개 파괴" },
  },

  // ---- 성직자 계열 ----
  cleric: {
    id: "cleric", name: "성직자", tier: 2, line: "holy", parent: "apprentice",
    weapon: "rpg:cleric_cross", skills: CLERIC_SKILLS, basic: false, corrupt: false, highMana: true, theme: "blue",
  },
  angel: {
    id: "angel", name: "천사", tier: 3, line: "holy", parent: "cleric",
    weapon: "rpg:angel_hands", skills: ANGEL_SKILLS, basic: false, corrupt: false, highMana: true, theme: "blue",
    req: { key: "light_angel", need: 64, label: "축복의 빛 1세트 바치기 (/rpg:bless)" },
  },
  // 미카엘: 천사 스킬 + 패시브 '불사'
  michael: {
    id: "michael", name: "미카엘", tier: 4, line: "holy", parent: "angel",
    weapon: "rpg:michael_wings", skills: ANGEL_SKILLS, basic: false, corrupt: false, highMana: true, theme: "blue",
    passive: "undying",
    req: { key: "light_michael", need: 192, label: "축복의 빛 3세트 바치기 (/rpg:bless)" },
  },
  fallen_angel: {
    id: "fallen_angel", name: "타락천사", tier: 4, line: "holy", parent: "angel",
    weapon: "rpg:fallen_angel_wings", skills: FALLEN_SKILLS, basic: false, corrupt: true, highMana: true, theme: "red",
    req: { key: "angel_villager_kills", need: 100, label: "천사 상태에서 주민 처치" },
  },
};

export const BASIC_JOB_IDS=['swordsman','mage','archer','fighter','cleric'];
export const NORMAL_JOB_IDS=Object.freeze([...BASIC_JOB_IDS,'sword_saint','fallen_sword_saint','sage','heavenly_archer','angel','michael','fallen_angel','fist_bone']);
for(const job of Object.values(JOBS))if(job.id!=='apprentice'&&!job.placeholder)job.hidden=!NORMAL_JOB_IDS.includes(job.id);

export const ALL_WEAPON_IDS = [
  ...Object.keys(STARTER_ITEMS),
  ...Object.values(JOBS).map((j) => j.weapon).filter(Boolean),
  ...Object.values(JOBS).map((j) => j.altWeapon).filter(Boolean),
];

// id("sword_saint") 또는 이름("검성")으로 직업 찾기
export function findJob(input) {
  if (input === undefined || input === null) return null;
  const s = String(input).trim();
  if (!s) return null;
  if (JOBS[s]) return JOBS[s];
  // 명령어에서 한글 직업명의 띄어쓰기를 생략해도 동일하게 인식합니다.
  // 예: /force 반물질의신, /rpg:force 반물질의 신
  const normalized = s.replace(/\s+/g, "");
  if (["타락검사", "타락 검사", "fallen_swordsman"].includes(normalized)) return JOBS.fallen_sword_saint;
  return Object.values(JOBS).find((j) => j.name.replace(/\s+/g, "") === normalized || j.id === normalized) ?? null;
}
```

## RPG_BP_v0578/scripts/data/skillGuide.js

```javascript
// Descriptions checked against the active runners, not unused historical skills.
export const LINE_NAMES={the_world:'더 월드',sword:'검사·검성·타락검성',mage:'마법사·현자',archer:'궁수·천궁',holy:'천사는 아군의 실제 HP 회복을 누적해 300마다 축복의 빛 1개를 받습니다. 자가 회복·초과 회복·자연 재생·외부 재생 포션은 제외하고 성직 재생 기술은 집계합니다. 길들인 아군 동물도 지원합니다. 성직자·천사·미카엘·타락천사',fist:'무투가·권골',berserker:'망나니·야수',assassin:'암살자·그림자 암살자',harpoon:'작살꾼·복수귀',blood:'혈술사',hacker:'해커',dragon:'용기사',frost:'냉기 능력자',fire:'화염술사',tech:'마법공학사',demon:'악마',celestial_sun:'해의 후예',celestial_moon:'달의 후예',chameleon:'카멜레온',war_god:'투신'};
export const LINE_NOTES={the_world:'해금 조건 미정인 근접 격투 히든 직업. 관리자 /rpg:force the_world로 시험합니다. 황금 화살은 전용 건틀릿 매개이며 손에서는 숨겨집니다. 시간 정지 중 피해는 종료 시 적용합니다. 사망·접속 종료·차원 이동 취소 시 누적 피해는 취소됩니다.',
 war_god:'무투가와 별개인 독립 근력 기반 히든 직업. 무투가(권골 제외)에서 레벨 40·5단계 이상 던전 솔로 클리어 조건. 검붉은 건틀릿을 사용합니다. 천멸격은 다음에 사용하는 공격 스킬 한 번의 피해를 3배로 만듭니다. 대상이 없어 빗나가면 소모하지 않습니다.',
 sword:'검성·타락검성은 휘두르기 기본 참격을 사용합니다. 검성의 섬멸은 타락 전직 시 사라지고 광분화로 교체됩니다. 타락검성은 검성의 무조건 상위호환이 아닙니다.',
 mage:'고마나 계열: 기본 MP 200＋바친 인챈트 책 1권당 MP 20. 현자로 전직하면 견습 파이어볼·라이트닝은 사용 목록에서 빠집니다. 다단 피해는 전탄 명중 상한과 실제 피격 무적을 구분해야 합니다.',
 archer:'기본 화살은 엔진 투사체 피해입니다. 천궁은 차징 없이 발사하며 실물 화살이 없으면 발당 고정 MP 20으로 회수 불가 마나 화살(기본 피해 6)을 발사합니다. 연사 5발은 MP 100＋연사 스킬 비용. 마나 화살에는 지능 할인이 없으며 천궁 고유 패시브라 카멜레온이 기술을 배워도 얻지 않습니다.',
 holy:'천사 계열 날개와 미카엘 불사는 직업 고유 기능입니다. 불사는 생존 HP가 6 이하일 때 최대 HP 회복·저항 V 4초·재생 III/화염 저항 5초, 재사용 60초이며 이미 사망한 캐릭터를 부활시키지는 않습니다. 광역 회복·보호·힘 버프는 자신과 파티·길드·팀 아군에게만 적용됩니다.',
 fist:'강펀치는 평타와 일부 물리 격투 기술을 강화합니다. 풍압은 마력 판정이며 힘 버프 피해 증가도 받지만 강펀치 횟수는 소비하지 않습니다. 낙하 피해 방지 등 일부 방어 이벤트는 실행 중인 Bedrock API의 지원 여부에 영향을 받습니다.',
 berserker:'망나니·야수는 처치 시 신속 III·재생 III를 5초 얻습니다. 카멜레온도 배운 참수·3연베기·야수 화염방사·대폭발을 전용 무기로 사용합니다. 광분화 학습 카멜레온의 처치 버프는 기존 동작대로 유지합니다.',
 assassin:'그림자 은신의 3배 기습과 표식의 1.3배가 함께 작동하면 조건을 만족하는 한 번의 피해가 3.9배입니다. 현재 entityAttack 원인의 스킬도 기습으로 판정될 수 있습니다. 카멜레온은 배운 그림자 은신의 유지·기습을 사용할 수 있습니다.',
 harpoon:'복수귀 순서: 딥 피어스 → 찢어내기 → 작살 → 돌진 → 영혼작살 → 잠수 → 영혼분리 → 처형. 공격 기술을 앞에, 버프·유틸을 뒤에 두고 처형을 마지막에 배치합니다. 새 두 공격은 짧은 쿨타임·일반 3 SP 학습·사용 횟수 제한 없음. 처형은 반경 4칸 기준입니다.',
 blood:'혁린약동·재는 사용 시 재생 II 1분을 부여합니다. 적박만 자체 흡혈하며 시전당 최대 6하트를 회복합니다.',
 hacker:'전직은 로토와 23회 대화 후 퀴즈를 맞힌 월드 최초 1명에게만 허용됩니다. 이후 정답자는 인챈트된 황금사과 1개를 받습니다. 레벨·엘리트 처치 조건은 없습니다. 재부팅과 계약 관련 밸런스는 이번 요청에서 유지했습니다. 재부팅은 현재 제외 목록에 없는 일부 궁극기와 재부팅 자체도 초기화합니다. 카멜레온의 학습 사용 횟수는 초기화하지 않습니다. EMP는 기술 봉인·마나 삭제가 아닙니다.',
 dragon:'용기사는 근접 3타마다 추가 피해 10＋HP 6 회복을 얻습니다. 카멜레온도 배운 삼조격·질주·브레스를 사용하지만 용기사의 근접 3타 패시브는 얻지 않습니다. 일부 방어 효과는 before-hurt API가 필요합니다.',
 frost:'동결은 둔화·무력화와 위치 복귀로 대상을 묶습니다. 절대영도의 단순 둔화와 다르며, 현재 CC 점감·보스별 동결 내성은 따로 없습니다.',
 fire:'화염술사는 상시 화염 저항 I. 화상 추가 피해는 화염 면역 적에게 약하지만 직접 기술 피해까지 전부 화염 피해인 것은 아닙니다. 화갑 반격은 카멜레온 학습자도 유지합니다.',
 tech:'마공학 감옥이 표식을 공급하고 구체·폭탄이 표식 대상을 추적합니다. 폭풍은 표식 없이도 지정 위치에 생성됩니다. 보조 기계팔은 마법공학사 전용 장착물입니다.',
 demon:'악마는 적 처치 시 HP 6 즉시 회복＋재생 II 5초를 얻습니다. 이 영혼 수확 패시브는 카멜레온에게 자동 승계되지 않습니다. 자가 계약과 적 대상 계약의 효과가 다릅니다.',
 celestial_sun:'해의 후예 전직은 천체 신전의 해 제단 4칸 안에서 낮에만 가능합니다(레벨 20·낮 적 60). /rpg:temple로 위치를 확인하세요. 빛 최대 저장량 500. 오버월드 낮·하늘이 열린 장소에서는 해당 빛 비용을 면제하고, 그 외에는 저장 빛을 소비합니다. MP는 별도로 듭니다. 충전도 해당 환경에서만 가능합니다. 축복의 상태이상 면역은 위치 고정/강제 이동까지 모두 해제하는 완전 CC 면역이 아닙니다.',
 celestial_moon:'달의 후예의 공격 기술은 근력·물리 피해 보정을 받습니다. 달의 후예 전직은 천체 신전의 달 제단 4칸 안에서 밤에만 가능합니다(레벨 20·밤 적 60). /rpg:temple로 위치를 확인하세요. 달빛 최대 저장량 500. 오버월드 밤·하늘이 열린 장소에서는 일반 기술 빛 비용을 면제합니다. MP는 별도입니다. 카멜레온이 차크람 분열을 배우면 버프 중 해당 기술을 재사용하여 전용 무기로 투척합니다.',
 chameleon:'기본 기술 2개는 무료. 선택 학습은 일반 3 SP·중요 9 SP·궁극기/카피 15 SP. 중요 기술 10회, 궁극기/카피 3회이며 다이아 1개로 각각 +5/+1회 충전합니다. 실제 5레벨 상승마다 궁극기 제외 랜덤 책 1권. 스킬 분해 시 일반 3·중요 9·궁극기/카피 15 SP를 환급하며 스킬북 기술도 대상입니다. 재학습해도 남은 사용 횟수·쿨타임은 이어집니다. 다른 직업 고유 패시브는 자동 승계되지 않습니다.',
};
export const SKILL_NOTES={
 world_muda:'전방 4칸 부채꼴, 0.25초 간격 8회. 기본 물리 피해 6씩 최대 48. 투신 난참과 같은 공격 판정·근력·장비 보정을 사용합니다.',
 world_finish:'전방 4칸 기본 물리 피해 30과 강한 밀치기·1초 둔화.',
 world_roller:'벽을 확인하며 전방 최대 3칸 이동한 뒤 기본 물리 피해 35. 블록을 파괴하지 않습니다.',
 world_high:'8초간 신속 II. 전용 화살 평타의 최소 간격이 0.5초에서 0.25초로 줄어듭니다. 다른 스킬 쿨타임·연타 간격은 유지합니다.',
 world_stop:'반경 20칸 적·투사체 최대 60개를 5초 정지. 보스 2.5초. 이동·시점 입력 차단과 위치 고정. 시전자 전용 공격의 피해는 정지 해제 순간에 적용합니다. 쿨 20초, 기본 MP 320·지능 보정 후에도 최소 300. 사망·접속 종료·차원 이동 시 즉시 해제하며 이때 누적 피해는 취소됩니다. 아군은 제외합니다.',

 war_god_punch:'청백색 부채꼴 충격파를 내뿜으며 전방 4칸·반폭 2칸에 기본 물리 피해 40. 밀치기와 짧은 둔화. 벽 너머 적은 제외합니다.',
 war_god_flurry:'청백색 충격파와 타격 섬광을 반복하며 전방 4칸에 0.25초 간격으로 6회, 기본 피해 10씩 명목 합계 60. 매 회차 범위를 다시 검사하므로 적이 벗어나면 남은 타격은 빗나갑니다.',
 war_god_kick:'아래에서 위로 솟는 청백색 발차기 궤적. 전방 6칸·반폭 2.5칸에 기본 물리 피해 48. 강한 밀치기와 1초 둔화. 블록은 파괴하지 않습니다.',
 war_god_kick_flurry:'발차기 궤적과 청백색 폭발을 교차시키며 전방 5.5칸에 0.4초 간격으로 4회, 기본 물리 피해 18씩 합계 72. 마지막 발차기에 강한 밀치기와 1초 둔화.',
 war_god_black_flash:'다음에 사용하는 투신 공격 스킬(폭멸타·난참·폭열권·파황각) 한 번의 피해를 3배로 강화합니다. 연타 스킬은 모든 타격에 3배가 적용됩니다. 평타는 강화되지 않으며, 대상이 없어 빗나간 스킬은 소비하지 않습니다. 이미 준비 중이면 다시 시전할 수 없습니다. 준비 상태와 쿨타임은 재접속해도 유지됩니다.',
 sword_spin:'시전자 주변 반경 4.5칸을 360도로 베어 기본 피해는 검사·타락검성·카멜레온 10, 검성 20(2배)과 밀치기. 광역 적에게 각각 한 번 피해를 줍니다.',
 sword_wave:'전방 사거리 18칸의 검기 투사체. 기본 피해는 검사·타락검성·카멜레온 14, 검성 21(1.5배). 폭 2.5칸으로 경로의 적을 공격하고 밀어냅니다.',
 sword_flame_spin:'약 1.4초 동안 회전하며 전진하는 불꽃 참격. 반경 3.4칸에 0.5초 간격으로 최대 3회 베기. 타격당 기본 피해 12.6(검성 18.9), 전부 맞으면 37.8(검성 56.7)과 4초 화상. 최대 돌진 거리 7칸. 점프 입력과 연동하는 공중 연출이 있습니다.',
 sword_kenho_rando:'큰 청록 초승달과 작은 보랏빛 참격이 겹치는 사거리 5.5칸의 세로 호 참격 3회. 각 기본 피해 12, 0.6초 간격이며 전부 맞으면 합계 36입니다.',
 sword_moon_dragon:'시전 시 바라본 방향의 전방 180도·반경 12칸에 0.6초 동안 큰 청록 초승달과 작은 보랏빛 달이 퍼집니다. 기본 물리 피해 28, 적마다 사용 1회당 한 번만 타격합니다. 높이 차 3칸·벽 가림을 검사하며 달의 개수로 피해가 중복되지 않습니다.',
 sword_annihilation:'전방 최대 15칸 이동하며 이동 경로 반경 4칸을 벱니다. 대상별 기본 피해 52를 한 번 적용하고 밀어냅니다. 검성 전용, 타락검성은 보유하지 않습니다.',
 sword_rush:'반경 10칸의 최근 타격 대상을 붙잡아 기본 피해는 검성 6씩 8회(합계 48), 타락검성·카멜레온은 4씩 8회(합계 32) 시도합니다. 연타 동안 저항 V를 얻어 공격과 방어를 함께 수행합니다.',
 fallen_sword_frenzy:'20초 동안 직접 준 피해의 30%를 흡혈합니다. 날개 강제 활성화는 원래 타락검성 직업에 제한되며 카멜레온에게 날개까지 자동 지급하지 않습니다.',
 mage_fireball:'사거리 40칸의 화염 투사체. 충돌 지점 반경 3칸에 기본 피해 6을 주는 견습·마법사 기술입니다.',
 mage_lightning:'시선 앞 사거리 24칸의 대상에 기본 피해 8. 현재 단일 번개 기술이며 유도탄 4발 기술은 봄바예입니다.',
 mage_flame_on:'시전자 주변 반경 15칸에 기본 피해 10과 화상 5초. 넓은 몰이용이며 직접 피해 효율은 낮습니다.',
 mage_thunder:'사거리 20칸 대상에게 번개를 3번, 0.4초 간격으로 시도합니다. 데이터 피해 25는 3회 합계입니다. 실제 벼락 소환 성공 시 벼락 추정 피해를 빼서 직접 피해를 계산합니다.',
 mage_ice_fire:'반경 10칸에 기본 피해 12와 둔화 VI를 4초 적용하는 광역 제어기입니다.',
 mage_gravity:'반경 10칸에 기본 피해 10과 둔화 IV를 5초 적용하고 아래 방향으로 계속 누릅니다. 중심으로 끌어모으는 블랙홀이나 위치 고정형 동결과는 다릅니다.',
 mage_bombaye:'주변 25칸에서 적을 탐색해 유도탄 4발을 발사합니다. 발당 14×0.3=4.2, 전탄 명목 합계 16.8. 비행 사거리 30칸이며 빗나가도 끝에서 작은 폭발을 만듭니다.',
 sage_meteor:'최근 마법 명중 대상이 60칸 안에 있으면 추적하고, 없으면 조준 지점에 낙하합니다. 준비 2초, 폭발 반경 8칸. 중심 기본 피해 58, 가장자리까지 선형으로 최대 50% 감소합니다.',
 sage_raiyoung:'사거리 35칸의 빠른 번개 구체. 기본 피해 28.8, 적중 위치의 폭발 연출과 번개 효과를 사용합니다.',
 sage_zero:'반경 20칸 적을 5초간 강한 둔화로 묶고, 시간이 끝나면 기본 피해 38.4. 위치 복귀로 강제 고정하는 냉기 직업의 동결과는 다릅니다.',
 sage_blackhole:'반경 12칸 블랙홀을 6초 유지해 적을 끌어당깁니다. 0.5초마다 기본 피해 약 4.1667, 전부 유지하면 합계 50.',
 sage_sphere:'사거리 32칸 큰 구체가 충돌하거나 사거리 끝에 도달하면 반경 7칸에 기본 피해 50와 밀치기.',
 sage_zoltraak:'전방 마법진에서 약 1초 준비 후 사거리 50칸·전체 명중 폭 10칸(중심에서 좌우 5칸)의 광선을 발사합니다. 기본 피해 96. 데이터 이름에 궁극기가 없지만 기존 궁극기로 분류하여 랜덤 책에서 제외합니다.',
 archer_airwalk:'지속 중 공중에서 점프 입력으로 다시 도약합니다. 견습 3초·궁수 8초·천궁 15초이며 비행을 무제한 상시 제공하지 않습니다.',
 archer_rapid:'일반 화살을 한 번에 견습 2발·궁수 3발·천궁 5발 발사합니다. 천궁의 마나 화살 모드에서는 5발 모두 발당 MP 20이 필요합니다. 실제 화살 사용 시 기존 한 묶음당 실물 1개 규칙을 유지합니다.',
 archer_fire:'이번 사격을 불 화살로 준비해 명중 대상에 5초 화상. 천궁 전직 시 폭탄 화살로 교체됩니다. 기본 화살 피해와 화상은 별도입니다.',
 archer_ice:'이번 사격 명중 대상에 둔화 III 5초와 추가 피해 3을 적용합니다.',
 archer_lightning:'이번 사격 명중 대상을 1초간 둔화·약화와 위치 복귀로 기절시킵니다.',
 archer_homing:'2분간 비행 중인 화살이 주변 24칸에서 적을 찾아 궤도를 수정합니다. 화살 자체 수명은 발사 후 5초입니다.',
 archer_bomb:'이번 사격 명중 대상에게 추가 피해 14. 폭발은 시각 효과이며 주변 전체 광역 피해나 블록 파괴는 없습니다.',
 archer_pierce:'사거리 38칸의 관통 궤적을 발사해 경로의 적에게 기본 피해 18. 실제 화살 아이템 대신 별도 광선 판정을 사용하는 기술입니다.',
 archer_judgement:'최근 피해를 준 살아 있는 적을 추적하는 전용 화살. 기본 피해 65를 한 번 주고 벽에 막히면 사라집니다. 표기 추적 사거리 96칸이며 최근 대상이 없으면 발동을 취소합니다.',
 archer_heavenly:'사거리 52칸의 관통 광선에 기본 피해 42. 첫 적중 위치에서 한 번만 반경 5칸 폭발을 만들고, 그 첫 대상 이외의 주변 적에게 추가 피해 18을 줍니다.',
 fist_strong:'30초 안의 다음 3회 공격을 2배로 강화합니다. 일부 물리 격투 기술도 횟수를 소비합니다. 풍압은 마법 기술로 변경되어 제외됩니다.',
 fist_flurry:'목표에게 접근하여 0.1초 간격으로 10회 공격합니다. 기본 피해 20은 10회 총합이며, 힘 효과와 강펀치는 별도로 반영합니다.',
 fist_smash:'1초 준비 후 전방 적에게 돌진해 기본 피해 28과 밀치기. 최대 탐색 사거리 32칸.',
 fist_wind:'전방 사거리 12칸·약 120도 부채꼴에 기본 마법 피해 20. 수평 밀치기 6.4·수직 1.8로 이전 값의 2배. 마력·마법 룬을 적용하고 힘 효과를 적용하지만 강펀치 횟수는 소비하지 않습니다.',
 fist_face:'반경 6칸 적을 시전자 정면으로 끌어오고 1초 기절시킵니다. 직접 피해는 0이며 지형·공간 검사 때문에 벽이나 낭떠러지를 강제로 관통해 끌지는 않습니다.',
 fist_stance:'자신에게 강한 둔화를 1초 적용하고 힘 II·저항 II를 15초 얻습니다. 준비 부담을 기존 3초에서 줄였습니다.',
 fist_finale:'반경 12칸 안의 최근 타격 대상을 고정해 상승·낙하 연출 후 고정 피해 60. 실제 대상을 높은 Y좌표로 옮기지는 않습니다. 근력·물리 룬에 따라 피해가 증가하지 않습니다.',
 holy_heal:'좌클릭한 생존 대상의 체력을 직업별 수치만큼 회복합니다. 힐은 예외적으로 적에게도 사용할 수 있지만, 적 회복은 아군 치유 기여·축복의 빛으로 계산하지 않습니다.',
 holy_shield:'반경 8칸에 흡수 V와 저항을 300초 부여합니다. 저항은 성직자 I·천사 II·미카엘 III. 자신과 같은 파티·길드·팀 아군에게만 적용합니다.',
 holy_sword:'반경 8칸 플레이어에게 성직자 힘 I·천사 II·미카엘 III를 300초 부여합니다. 자신과 같은 파티·길드·팀 아군에게만 적용합니다.',
 holy_orb:'아군 플레이어 한 명에게 부착해 재생·신속·흡수를 유지하고 시전자는 허기 II를 받습니다. 성직자 재생/신속 II·흡수 VIII, 천사 III/IX, 미카엘 IV/X. 거리 30칸·차원·생존·아군 조건을 벗어나면 해제됩니다. 카멜레온 학습자도 성직자 단계로 유지합니다.',
 holy_pray:'게임 시간 하루 한 번, 10초 실명·강한 둔화 후 축복의 빛을 성직자 1·천사 2·미카엘 3개 지급합니다. 전투 회복기가 아닌 제작 재료 획득기입니다.',
 angel_bless:'반경 12칸 플레이어 HP 20 회복, 재생 III·저항 II·신속 II 300초와 시듦·독·약화 제거. 자신과 같은 파티·길드·팀 아군만 지원합니다.',
 angel_zone:'반경 8칸에 15초 영역. 아군에게 1초마다 HP 2 회복, 최소 재생 V·저항 I 유지. 적에게 둔화 II. 쿨타임 150초. 적에게 지원 버프를 주지 않습니다.',
 fallen_sacrifice:'적을 좌클릭해 기본 피해 16을 줍니다. 추가 회복이나 흡혈 효과는 없습니다.',
 fallen_sword:'사거리 18칸 관통 검기로 기본 마법 피해 38, 시듦 II 5초와 밀치기.',
 fallen_shield:'5초 동안 저항 III. 받은 피해의 100%를 공격자에게 반사하고 시듦 I 3초.',
 fallen_curse:'시선 앞 사거리 30칸 적에게 기본 피해 10, 시듦 III·약화 II·둔화 II를 10초 적용합니다.',
 fallen_orb:'좌클릭으로 적에게 부착하거나 우클릭으로 구슬을 발사합니다. 기본 마법 피해 16, 최초 부착 시 시듦 III 10초. 부착 중 받는 피해 +35%. 시전자와 30칸 이상 떨어지거나 사망·차원 변경 시 해제. 시전자당 대상 하나, 중첩 불가.',
 fallen_zone:'시전 지점 반경 9칸에 15초 저주 영역. 1초마다 기본 피해 3과 시듦 I·둔화 II. 이론상 지속 피해 45이며 영역 체류와 피격 무적의 영향을 받습니다.',
 fallen_end:'4초 준비 중 시전자 저항 V, 주변 반경 25칸 적에게 둔화 I를 반복 적용합니다. 범위는 시전자를 따라 이동하고 마지막에 기본 피해 60·시듦 III 5초·밀치기. MP 800으로 조정했습니다.',
 berserker_charge:'전방 17칸에서 대상을 찾아 돌진·기본 피해 14. 벽 가까운 적에게 2초 기절과 추가 피해 배율 표식을 적용합니다.',
 berserker_slam:'반경 5칸에 기본 물리 폭발 피해 18을 먼저 주고 생존한 적을 위로 최대 10칸 띄웁니다. 천장을 통과하지 않으며 보스 띄우기는 절반.',
 berserker_execute:'다음 기본 근접 공격 ×4. 적의 타격 직전 체력이 최대 체력의 15% 미만이면 처형합니다. 정확히 15%는 즉사 조건이 아닙니다. 망나니·야수의 원래 무기와 학습한 카멜레온 무기로 사용.',
 beast_berserk:'20초 동안 허기 III·힘 IV·저항 II·신속 II. 야수는 추가 피해 흡수 IV와 흡혈 15%를 받습니다. 흡수의 룬과 흡혈 비율은 높은 값 하나를 사용합니다. 카멜레온은 기존 공통 버프를 받습니다.',
 berserker_triple_slash:'짧게 전진하며 기본 피해 12씩 3회, 0.5초 간격 참격. 명목 합계 36. 카멜레온 학습자도 3회의 후속 참격을 사용합니다.',
 beast_flame_burst:'전방으로 화염 구체 3발을 순차 발사. 발당 기본 물리 피해 15, 최대 합계 45와 3초 화상.',
 beast_grand_explosion:'약 1초 후 반경 15칸 폭발로 기본 피해 70. 블록을 파괴하는 실제 폭발이 아닌 스킬 피해·파티클입니다. 카멜레온 학습자도 발동합니다.',
 assassin_spin:'약 3칸 이동하며 반경 3칸에 기본 피해 24의 회전 참격. 벽 안으로 이동하지 않습니다.',
 assassin_dagger_wave:'사거리 22칸 단검 투사체. 기본 피해 25와 2초 둔화.',
 assassin_pierce:'전방 4.5칸 적에게 기본 피해 15와 10초 회복 봉쇄. 5틱마다 HP를 기록 상한 이하로 되돌리며 이후 피해를 받으면 상한도 내려갑니다.',
 assassin_stealth:'전방 12칸의 적 그림자로 이동하고 은신을 유지합니다. 다음 entityAttack 원인 피해를 3배로 증폭한 뒤 은신을 해제합니다. 적이 없으면 발동하지 않습니다.',
 assassin_mark:'전방 24칸 적에게 10초 동안 받는 피해 30% 증가 표식. 현재 추가 피해는 실제 받은 피해를 기준으로 HP를 직접 줄여 적용합니다.',
 shadow_assassinate:'25칸 안의 표식 대상이 필요합니다. 3초 동안 적을 따라가며 보호·은신 연출 후 기본 피해 38을 주고 안전한 주변 위치로 빠집니다.',
 revenant_deep_pierce:'전방 5.5칸·반폭 0.9칸을 찔러 기본 물리 피해 18. 벽에 가리지 않은 경로의 적을 관통하며 별도 대상 지정이 필요 없습니다. MP 18·쿨 2.5초.',
 revenant_rend:'전방 4.5칸·반폭 2.4칸을 베어 기본 물리 피해 16. 적중한 적에게 1초마다 3씩 세 번 추가해 명목 합계 25. 같은 시전자의 출혈은 중첩하지 않고 갱신합니다. MP 24·쿨 4초.',
 harpoon_throw:'사거리 30칸 작살 투사체에 기본 피해 12. 끌어오기 효과는 복수귀의 영혼작살에 있습니다.',
 harpoon_dive:'작살꾼은 5초 투명화·신속 III, 복수귀는 10초 투명화·신속 IV·재생 II를 얻습니다.',
 harpoon_charge:'전방 길이 5칸·폭 2.5칸의 직사각형 범위를 공격하며 5칸 이동합니다. 범위 안 적에게 기본 피해 28. 현재 지면·벽 검사가 부족해 이동 시 주의해야 합니다.',
 soul_harpoon:'사거리 34칸에 기본 마법 피해 32의 영혼 작살. 적중 대상을 시전자 쪽으로 끌어옵니다.',
 soul_separation:'바닥 조건 없이 전방 7칸으로 이동한 뒤 1초 후 귀환 경로를 따라 반경 3칸에 걸린 적을 3초 고정·기절시킵니다. 직접 피해보다 접근·광역 제어용입니다. 경로와 도착 위치의 벽은 통과하지 않습니다.',
 revenant_execution:'시전자 반경 4칸 안에서 HP 25% 이하 적들을 처형합니다. 성공하면 재생 III 5초·쿨 5초. 대상이 없으면 같은 반경의 최근 타격/조준 적에게 기본 피해 50, 쿨 60초. 직접 kill 처치의 보상 귀속은 실제 엔진 확인이 필요합니다.',
 blood_spear:'10회 공격 가능한 피의 창. 손에 들고 사용하면 6칸 찌르기. 기본 마법 피해 14·시듦 I 3초.',
 blood_orb:'쿨타임 5초. 사거리 30칸 구체. 직접 피해 18, 반경 2.5칸 파편 피해 8.',
 blood_supernova:'정면 최대 3칸 앞에 혈액 구체를 소환하고 0.75초 뒤 폭발시킵니다. 반경 5칸 기본 마법 피해 30, 쿨타임 15초.',
 blood_mark:'4칸 이내 근거리 기본 물리 피해 17, 원거리 사거리 27칸 기본 피해 15.',
 blood_awaken:'사용 시 재생 II 1분. 활성화 중 힘 III·저항 III·허기 III 유지. 1분 후 시듦 V. 다시 사용하면 해제합니다.',
 blood_bind:'반경 16칸 기본 마법 피해 10과 5초 속박. 실제로 준 피해만큼 시전당 최대 6하트 회복. 보스 속박은 기존 제어 저항을 따릅니다.',
 hacker_hack:'우클릭은 32칸 안 최근 타격 적에게 약화 IV·둔화 III·실명 3초. 아군을 8칸 안에서 좌클릭하면 재생 III·저항 III·포화를 20초 지원합니다. 자가 지원 해킹은 불가합니다.',
 hacker_dummy_data:'사거리 20칸 적을 자동 탐색하는 유도 패킷. 기본 마법 피해 26, 추적 수명 2.5초.',
 hacker_reboot:'반경 30칸의 자신·길드·파티·팀 아군 일반 기술 쿨타임을 초기화합니다. 기존 제외 목록에 없는 일부 궁극기와 재부팅 자체도 초기화합니다. 카멜레온 사용 횟수는 복구하지 않습니다.',
 hacker_stealth:'투명화·신속·허기 효과를 유지하는 토글 은신입니다. 그림자 암살자의 다음 공격 3배 기습 기능은 없습니다. 다시 사용하면 해제합니다.',
 hacker_copy:'현재 등록된 공격·제어 궁극기 중 실행 가능한 기술을 무작위로 발동합니다. 순수 버프 궁극기(다음 평타를 강화하는 천멸격 포함)는 제외합니다. 공격과 버프가 함께 있는 궁극기는 포함하며, 표식·대상·빛 등 원래 조건을 충족해야 합니다.',
 hacker_ddos:'시전자 반경 10칸 적들에게 기본 마법 피해 30. 마나를 삭제하거나 기술을 봉인하는 효과는 없습니다.',
 hacker_emp:'시전 지점 반경 30칸에 5초 영역. 적에게 약화 IV·둔화 III·실명을 갱신하고, 진입한 아군에 한 번씩 재생 III·저항 III 20초. 스킬 사용 금지 효과는 없습니다.',
 dragon_triple_strike:'10초 안의 다음 3타에 추가 피해 12씩, 세 번째에 1초 기절을 준비합니다. 카멜레온도 배운 기술의 후속 강화 3타를 사용할 수 있습니다.',
 dragon_gaze:'사거리 28칸 투사체로 기본 피해 20. 용기사 표식 후속 이벤트가 작동하면 10초 받는 피해 +30%; 해당 이벤트의 직업·API 제한을 확인해야 합니다.',
 dragon_charge:'사거리 26칸에 창을 던지고 닿은 엔티티 또는 벽 직전으로 이동합니다. 적에게 기본 물리 피해 18, 아군은 이동 목표로만 사용합니다. 음수 Y에서도 작동하며 바닥·안전 지형을 요구하지 않습니다.',
 dragon_sprint:'5초 동안 전방으로 초당 21칸 질주합니다. 서로 다른 적에게 최초 충돌 시 14×1.5^(누적 적 수−1) 기본 물리 피해. 적 수에 따른 상한은 없으며 카멜레온 학습자도 발동합니다. 음수 Y·공중에서도 작동하고 벽에서 정지합니다.',
 dragon_domain:'반경 7칸 수룡 영역을 10초 전개합니다. 진입 기본 피해 20과 주기 피해 8, 아군 저항 V. 카멜레온 학습자도 전개합니다. 외부 투사체 차단은 before-hurt API 지원 여부에 제한됩니다.',
 dragon_breath:'사거리 28칸의 용 브레스 투사체. 기본 피해 24·화상 4초. 카멜레온 학습자도 발사·후속 명중 효과를 사용합니다.',
 frost_shard:'사거리 32칸 서리 투사체. 기본 피해 14와 4초 둔화.',
 frost_grasp:'정면 7칸의 벽에 가리지 않은 적 한 명에게 기본 피해 12와 2초 동결. 시야 밖 또는 벽 뒤 적은 잡지 않습니다.',
 frost_barrage:'사거리 24칸 안의 적을 향해 얼음 유도탄 5발을 발사합니다. 날아가는 동안 이동 중인 적을 추적하고 여러 적이 있으면 분산합니다. 발당 기본 마법 피해 7·전탄 명목 합계 35와 둔화 II 2.5초. 벽에 닿거나 비행 거리가 끝나면 소멸합니다.',
 frost_step:'바닥·안전 지형 조건 없이 전방 6칸으로 이동하고 신속 III·저항 I를 5초 얻습니다.',
 frost_field:'시전 지점 반경 7칸에 5초 결계. 적에게 처음 닿을 때 기본 피해 10, 머무는 동안 둔화 III를 갱신합니다. 매번 10씩 반복 피해를 주지는 않습니다.',
 frost_great_release:'0.4초 준비 후 전방 10칸의 적에게 돌진해 기본 마법 피해 42와 3초 동결.',
 frost_absolute:'반경 12칸 적에게 기본 마법 피해 38과 5초 위치 고정형 동결. 보스 제어 시간은 절반.',
 fire_ignite:'시전자 주변 반경 7칸에 기본 피해 15와 5초 화상.',
 fire_pillar:'사거리 22칸의 조준 지점 반경 3칸에 기본 피해 22·위로 밀치기·3초 화상.',
 fire_wheel:'최대 8칸 전진하는 화염바퀴로 경로의 적에게 기본 피해 18과 4초 화상.',
 fire_armor:'10초 힘 II·저항 II. 3.2칸 안의 근접 공격자에게 3초 화상 반격. 상시 화염 저항은 실제 화염술사의 별도 패시브입니다.',
 fire_emperor:'반경 12칸에 기본 피해 36·5초 화상. 5초간 주변 영역이 매초 기본 피해 4를 시도하고, 자신은 힘 III·저항 II를 15초 얻습니다.',
 tech_ray:'사거리 26칸 관통 광선으로 대상별 기본 마법 피해 18과 10초 표식을 부착합니다. 지면에 4초 에너지 흔적을 남겨 주기적으로 기본 피해 4.',
 tech_prison:'사거리 16칸의 표식 적을 우선하고, 없으면 가장 가까운 적의 위치에 반경 4칸 감옥을 6초 설치합니다. 적이 없으면 발동하지 않습니다. 내부 적에게 둔화 IV와 10초 표식을 반복 갱신합니다.',
 tech_sphere:'36칸 안의 표식 적을 찾아 유도 구체로 기본 피해 22. 표식이 없으면 발동을 취소합니다.',
 tech_bomb:'32칸 안의 표식 대상에 폭탄을 부착합니다. 약 2초 뒤 대상에 기본 피해 30, 반경 2.6칸의 다른 적에 피해 9. 표식 없이 발동하지 않습니다.',
 tech_storm:'반경 5칸에 10초 폭풍을 생성해 0.5초마다 기본 피해 5를 시도합니다. 명목 합계 100. 36칸 안 표식 대상을 추적하고 표식이 끝나면 그 자리에 남습니다.',
 demon_fear:'사거리 26칸 공포 투사체로 기본 피해 16, 둔화 III·약화 II·실명 4초와 밀치기. 몬스터의 완전한 강제 도주와 같지는 않습니다.',
 demon_vampirism:'반경 10칸 안의 모든 적에게 빨대를 연결하는 영혼 궤적. 각각 기본 마법 피해 12, 적중 수만큼 재생 단계(최대 50)를 7초 얻습니다. 빨대는 최대 7초 표시되며 추가 지속 피해는 없습니다. 아군·사망·차원 변경·반경 이탈 시 해당 연결은 종료합니다.',
 demon_reap:'전방 사거리 4.6칸의 낫 참격으로 기본 물리 피해 36.',
 demon_contract:'우클릭 자신: HP 10을 내고 저항 III·흡수 X 120초. HP가 10 이하면 취소. 좌클릭 적: 고정 피해 14를 주지만 적도 힘 III를 5초 얻습니다. 이번 버전에서 수치·동작을 유지했습니다.',
 demon_storm:'자신의 디버프를 정화하고 반경 10칸 공포 폭풍을 10초 유지하며 초당 기본 피해 6을 시도합니다. 전부 적중할 때 명목 합계 60입니다.',
 demon_domain:'반경 7.5칸에서 시전자와 선택한 적 하나를 남겨 10초 격리합니다. 나머지 살아 있는 엔티티를 밖으로 밀고 재진입을 막으며 선택 적에 허기 V. 단순 광역 피해기가 아닙니다.',
 sun_charge:'오버월드 낮·열린 하늘에서 햇빛 35를 충전합니다. 최대 500. 자연 충전은 기존 초당 4 유지. 마나 충전과 별도입니다.',
 sun_slash:'7초 안의 다음 전용 도끼 기본 타격을 총 4배로 강화하고 둔화 III 1초. 저장 햇빛 비용 10, 적절한 환경에서는 빛 비용 면제.',
 sun_blessing:'20초 저항 II·흡수 IV, 최초 한 번 받은 공격의 100% 반사. 환경 비용 면제가 없으면 햇빛 15 소비.',
 sun_divine_blessing:'디버프 제거와 3분 부정 상태이상 면역, 처음 3초는 공격 피격 무적시간 우회. 저장 햇빛 비용 20. 위치 고정형 CC를 전부 해제하지는 않습니다.',
 sun_meteor_drive:'전방 22칸 적을 찾아 마법 광선 피해 14 후 짧게 돌진해 물리 피해 26. 명목 합계 40, 저장 햇빛 비용 15.',
 sun_radiant_beam:'1초 준비 후 바라보는 방향으로 10칸 레이저를 5초 발사합니다. 발사 중 시선을 돌리거나 위아래를 보면 광선과 타격 방향도 따라 바뀝니다. 매 틱(정상 20 TPS 기준 초당 20회) 기본 마법 피해 1·명목 합계 100, 피격 무적 시간 우회와 화상·약화 II. 저장 햇빛 비용 20.',
 sun_descent:'반경 12칸에 기본 마법 피해 20, 자신에게 힘 III·재생 II·포화·흡수 X를 15초. 저장 햇빛 비용 25.',
 moon_charge:'오버월드 밤·열린 하늘에서 달빛 35를 충전합니다. 최대 500. 자연 충전은 기존 초당 4 유지. 마나 충전과 별도입니다.',
 moon_slash:'전방 사거리 5.5칸 부채꼴에 기본 물리 피해 30. 근력·물리 장비와 힘 버프 보정. 저장 달빛 비용 10.',
 moon_moonlight:'최대 HP의 50%를 회복하고 보유 달빛의 30%를 소비합니다. 마력 배율로 늘리는 일반 힐과 별도 계산입니다. 야외 밤이 아니면 저장 달빛이 있어야 하며 최대 HP에서는 취소합니다.',
 moon_homing:'주변 10칸의 적을 유도 차크람으로 추적해 기본 물리 피해 28을 한 번 줍니다. 수명 약 2.25초. 저장 달빛 비용 12. 일반 무기 투척과 별도.',
 moon_chakram_split:'60초 동안 차크람 투척마다 소형 차크람 3개(각각 기본 물리 피해 4, 근력·물리 장비 보정)를 추가합니다. 카멜레온은 켜자마자 투척하고, 효과 중 같은 기술 재사용으로 반복 투척합니다. 투척 간격 0.75초이며 반복 투척은 추가 MP·학습 횟수를 쓰지 않습니다. 저장 달빛 비용 15.',
 moon_veil:'20초 투명화·신속 IV. 저장 달빛 비용 15. 그림자 은신의 3배 기습 효과는 없습니다.',
 moon_fullwave:'반경 11칸에 기본 물리 피해 50과 둔화 III 3초. 저장 달빛 비용 25.',
 chameleon_crimson_dash:'흑염 대검에서 5초 신속 V와 접촉 고정 피해 20. 이 기술로 처치하면 연속 재사용 기회를 얻고 최대 3연속, 마지막은 20초 쿨. 일반 쿨은 7초입니다.',
 chameleon_ground_slam:'백월의 낫에서 반경 4.6칸으로 지면 충격파. 기본 피해 18이며 시각 파동 7개를 피해 18×7로 계산하지 않습니다. MP 무료·쿨 7초.',
};
```

## RPG_BP_v0578/scripts/data/skills.js

```javascript
// 스킬 데이터. cooldown 은 틱(20틱 = 1초), mana 는 소모 마나, damage 는 체력 수치(하트 1칸 = 2).
//   maxTier : 이 차수까지만 쓰는 스킬 (마법사 견습 스킬은 3차 전직 시 목록에서 빠짐)
// 실제 동작은 skills/ 폴더의 같은 id 함수가 담당합니다.
export const SKILLS = {
  world_muda:{id:'world_muda',name:'무다!×8',line:'the_world',mana:65,cooldown:140,range:4,width:2,hits:8,interval:5,damagePerHit:6,kick:false},
  world_finish:{id:'world_finish',name:'무다!!!',line:'the_world',mana:70,cooldown:180,range:4,width:1.8,hits:1,interval:5,damagePerHit:30,kick:true},
  world_roller:{id:'world_roller',name:'로드로러다!!',line:'the_world',mana:90,cooldown:240,range:4,width:2,hits:1,interval:5,damagePerHit:35,kick:true},
  world_high:{id:'world_high',name:'High한 기분',line:'the_world',mana:80,cooldown:400,duration:160},
  world_stop:{id:'world_stop',name:'더 월드',line:'the_world',mana:320,cooldown:400,duration:100,radius:20,ultimate:true},

 war_god_punch:{id:'war_god_punch',name:'폭멸타',line:'war_god',mana:60,cooldown:120,damagePerHit:40,hits:1,interval:0,range:4,width:2,kick:false},
 war_god_flurry:{id:'war_god_flurry',name:'난참',line:'war_god',mana:120,cooldown:200,damagePerHit:10,hits:6,interval:5,range:4,width:2,kick:false},
 war_god_kick:{id:'war_god_kick',name:'폭열권',line:'war_god',mana:90,cooldown:160,damagePerHit:48,hits:1,interval:0,range:6,width:2.5,kick:true},
 war_god_kick_flurry:{id:'war_god_kick_flurry',name:'파황각',line:'war_god',mana:160,cooldown:320,damagePerHit:18,hits:4,interval:8,range:5.5,width:2.5,kick:true},
 war_god_black_flash:{id:'war_god_black_flash',name:'천멸격 (궁극기)',buffOnlyUltimate:true,line:'war_god',mana:300,cooldown:1800,ultimate:true},
  // 카멜레온 흑염 대검 전용: 충돌 20 고정 피해 / 처치 시 연속 재사용 최대 3회.
  chameleon_crimson_dash: {
    id: "chameleon_crimson_dash", name: "적섬 돌진", line: "chameleon",
    mana: 0, cooldown: 140, damage: 20, duration: 100,
    speedAmplifier: 4, maxChain: 3, finishCooldown: 400, collisionRadius: 1.2,
  },
  chameleon_ground_slam: {
    id: "chameleon_ground_slam", name: "지면강타", line: "chameleon",
    mana: 0, cooldown: 140, damage: 18, radius: 4.6, pulses: 7, pulseStep: 0.34,
  },
  // 해의 후예: 유저 지정 기술. 지정 없는 수치는 임시 밸런스.
  sun_charge: {id:"sun_charge",name:"햇빛 충전",line:"celestial_sun",mana:0,cooldown:100,restore:35},
  sun_slash: {id:"sun_slash",name:"일륜참",line:"celestial_sun",mana:65,cooldown:160,damageMultiplier:4,slowTicks:20,slowAmp:2},
  sun_blessing: {id:"sun_blessing",name:"해의 가호",line:"celestial_sun",mana:110,cooldown:600,duration:400},
  // 해의 축복: 3분(3600틱)간 부정적 상태이상 면역, 최초 3초(60틱) 공격 무적시간 우회.
  sun_divine_blessing: {id:"sun_divine_blessing",name:"해의 축복",line:"celestial_sun",mana:155,cooldown:4200,immunityTicks:3600,pierceTicks:60},
  sun_meteor_drive: {id:"sun_meteor_drive",name:"메테오드라이브",line:"celestial_sun",mana:125,cooldown:300,range:22,beamDamage:14,impactDamage:26},
  sun_radiant_beam: {id:"sun_radiant_beam",name:"방사열선",line:"celestial_sun",mana:180,cooldown:600,chargeTicks:20,duration:100,range:10,damagePerPulse:1,pulsesPerSecond:20},
  sun_descent: {id:"sun_descent",name:"일륜강림 (궁극기)",line:"celestial_sun",mana:320,cooldown:1200,damage:20,radius:12,buffTicks:300,ultimate:true},
  // 달의 후예: 이전 버전 스킬 복원.
  moon_charge: {id:"moon_charge",name:"달빛 충전",line:"celestial_moon",mana:0,cooldown:100,restore:35},
  moon_slash: {id:"moon_slash",name:"초승달 참격",line:"celestial_moon",mana:65,cooldown:160,damage:30,range:5.5},
  moon_moonlight: {id:"moon_moonlight",name:"달빛",line:"celestial_moon",mana:250,cooldown:1200,healRatio:0.5,lightRatio:0.30},
  moon_homing: {id:"moon_homing",name:"월광 차크람",line:"celestial_moon",mana:75,cooldown:100,damage:28,range:10},
  // 달의 차크람을 실제 투척할 때마다 작은 차크람 3개를 추가 발사한다. 60초 유지.
  moon_chakram_split: {id:"moon_chakram_split",name:"차크람 분열",line:"celestial_moon",mana:90,cooldown:1500,duration:1200,smallCount:3,smallDamage:4},
  moon_veil: {id:"moon_veil",name:"밤의 장막",line:"celestial_moon",mana:95,cooldown:450,duration:400},
  moon_fullwave: {id:"moon_fullwave",name:"만월의 파동 (궁극기)",line:"celestial_moon",mana:310,cooldown:1200,damage:50,radius:11,slowTicks:60,ultimate:true},

  // ---------- 악마 (임시 피해량·마나·쿨타임) ----------
  demon_fear: { id:"demon_fear", name:"공포", line:"demon", mana:60, cooldown:160, damage:16, range:26, speed:1.25, fearTicks:80 },
  demon_vampirism: { id:"demon_vampirism", name:"흡혈", line:"demon", mana:90, cooldown:240, damage:12, radius:10, regenerationTicks:140, maxLevel:50 },
  demon_reap: { id:"demon_reap", name:"수확", line:"demon", mana:55, cooldown:120, damage:36, range:4.6 },
  demon_contract: { id:"demon_contract", name:"계약 (우클릭 자신 / 좌클릭 적)", line:"demon", trigger:"hybrid", mana:80, cooldown:400, healthCost:10, enemyDamage:14, enemyStrengthTicks:100, buffTicks:2400 },
  demon_storm: { id:"demon_storm", name:"공포의 폭풍 (궁극기)", line:"demon", mana:280, cooldown:1200, radius:10, duration:200, damagePerSecond:6, ultimate:true },
  demon_domain: { id:"demon_domain", name:"영역전개", line:"demon", mana:240, cooldown:900, radius:7.5, duration:200, hungerAmplifier:4, hungerRefresh:50 },

  // 마법공학사: 임시 피해량/마나/쿨타임. 사용자 요구는 행동/효과를 우선 적용.
  tech_ray: { id:"tech_ray",name:"마공학 광선",line:"tech",mana:90,cooldown:160,damage:18,range:26,groundTicks:80 },
  tech_prison: { id:"tech_prison",name:"마공학 감옥",line:"tech",mana:125,cooldown:340,radius:4,castRange:16,duration:120 },
  tech_sphere: { id:"tech_sphere",name:"마공학 구체",line:"tech",mana:70,cooldown:140,range:36,damage:22,speed:2.25 },
  tech_bomb: { id:"tech_bomb",name:"마공학 폭탄",line:"tech",mana:95,cooldown:220,range:32,damage:30,splashRadius:2.6,splashDamage:9,fuseTicks:40 },
  tech_storm: { id:"tech_storm",name:"마공학 폭풍 (궁극기)",line:"tech",mana:340,cooldown:1200,radius:5,followRange:36,duration:200,damagePerPulse:5,ultimate:true },
  // ---------- 암살자 / 그림자 암살자 (마나·미지정 쿨타임은 임시 밸런스) ----------
  assassin_spin: { id:"assassin_spin",name:"회전베기",line:"assassin",mana:60,cooldown:180,teleport:3,radius:3,damage: 24 },
  assassin_dagger_wave: { id:"assassin_dagger_wave",name:"몰아치는 단검",line:"assassin",mana:70,cooldown:160,damage:25,range:22,speed:1.45,hitRadius:1.1,slowTicks:40 },
  assassin_pierce: { id:"assassin_pierce",name:"피어스",line:"assassin",mana:65,cooldown:240,damage:15,range:4.5,noHealTicks:200 },
  assassin_stealth: { id:"assassin_stealth",name:"은신",line:"assassin",mana:70,cooldown:400,range:12 },
  assassin_mark: { id:"assassin_mark",name:"표식",line:"assassin",mana:75,cooldown:280,range:24,duration:200,damageMultiplier:1.3 },
  shadow_assassinate: { id:"shadow_assassinate",name:"그림자 암살 (궁극기)",line:"assassin",mana:280,cooldown:1000,damage:38,delay:60,range:25,ultimate:true },

  // ---------- 히든 직업 화염술사: 화염 저항 I 상시 패시브, 액티브 5종 ----------
  // 단위: 20틱=1초, 피해량은 체력 수치. 임시 밸런스.
  fire_ignite: { id:"fire_ignite", name:"자연발화", line:"fire", mana:55, cooldown:120, damage:15, radius:7, burnSeconds:5 },
  fire_pillar: { id:"fire_pillar", name:"불기둥", line:"fire", mana:110, cooldown:320, damage:22, range:22, radius:3, launch:1.0, burnSeconds:3 },
  fire_wheel: { id:"fire_wheel", name:"화염바퀴", line:"fire", mana:95, cooldown:220, damage:18, range:8, speed:1.3, wheelRadius:.75, hitRadius:1.5, burnSeconds:4 },
  fire_armor: { id:"fire_armor", name:"화갑", line:"fire", mana:100, cooldown:500, duration:200 },
  fire_emperor: { id:"fire_emperor", name:"염제 (궁극기)", line:"fire", mana:300, cooldown:1200, damage:36, radius:12, burnSeconds:5, zoneTicks:100, zoneDamage:4, buffTicks:300, ultimate:true },

  // ---------- 히든 냉기 능력자 (임시 밸런스: 피해는 체력 수치) ----------
  frost_shard: { id:"frost_shard",name:"서리탄",line:"frost",mana:35,cooldown:60,damage:14,range:32,speed:1.65,slowTicks:80 },
  frost_grasp: { id:"frost_grasp",name:"빙결의 손",line:"frost",mana:55,cooldown:160,damage:12,range:7,freezeTicks:40 },
  frost_barrage: { id:"frost_barrage",name:"빙주난사",line:"frost",mana:95,cooldown:240,shots:5,damage:7,range:24,speed:1.8,spread:.16 },
  frost_step: { id:"frost_step",name:"빙하보",line:"frost",mana:55,cooldown:200,distance:6,buffTicks:100 },
  frost_field: { id:"frost_field",name:"설한결계",line:"frost",mana:120,cooldown:400,radius:7,duration:100,damage:10 },
  frost_great_release: { id:"frost_great_release",name:"대빙해소",line:"frost",mana:100,cooldown:280,range:10,chargeTicks:8,dashSpeed:1.8,damage:42,freezeTicks:60 },
  frost_absolute: { id:"frost_absolute",name:"영점빙옥 (궁극기)",line:"frost",mana:300,cooldown:900,radius:12,damage:38,freezeTicks:100,ultimate:true },
  // ---------- 작살꾼 / 복수귀 ----------
  harpoon_throw: { id:"harpoon_throw",name:"작살",line:"harpoon",mana:50,cooldown:160,damage:12,range:30,speed:1.35 },
  harpoon_dive: { id:"harpoon_dive",name:"잠수",line:"harpoon",mana:70,cooldown:400,duration:100 },
  harpoon_charge: { id:"harpoon_charge",name:"돌진",line:"harpoon",mana:100,cooldown:300,damage:28,length:5,width:2.5 },
  soul_harpoon: { id:"soul_harpoon",name:"영혼작살",line:"harpoon",mana:75,cooldown:200,damage: 32,range:34,speed:1.45 },
  soul_separation: { id:"soul_separation",name:"영혼분리",line:"harpoon",mana:150,cooldown:600,range:7,delay:20,radius:3,stun:60 },
  revenant_deep_pierce: {id:'revenant_deep_pierce',name:'딥 피어스',line:'harpoon',mana:18,cooldown:50,damage:18,range:5.5,halfWidth:.9},
  revenant_rend: {id:'revenant_rend',name:'찢어내기',line:'harpoon',mana:24,cooldown:80,damage:16,range:4.5,halfWidth:2.4,bleedDamage:3,bleedHits:3,bleedInterval:20},
  revenant_execution: { id:"revenant_execution",name:"처형 (궁극기)",line:"harpoon",mana:200,cooldown:1200,successCooldown:100,failDamage:50,healthThreshold:.25,radius:4,ultimate:true },

  // ---------- 혈술사: 혈액 기반 마법 (임시 밸런스, 단위는 게임 틱) ----------
  blood_spear: { id:"blood_spear",name:"피의 창 제작",line:"blood",mana:40,cooldown:100,damage: 14,range:6,durability:10,witherTicks:60 },
  blood_orb: { id:"blood_orb",name:"혈성뢰",line:"blood",mana:75,cooldown:100,damage:18,splashDamage:8,burstRadius:2.5,range:30,speed:1.2 },
  blood_supernova: { id:"blood_supernova",name:"초신성",line:"blood",mana:120,cooldown:300,damage:30,burstRadius:5,forward:3,detonateTicks:15 },
  blood_mark: { id:"blood_mark",name:"혈인 (근거리·원거리)",line:"blood",mana:60,cooldown:120,meleeRange:4,meleeDamage:17,rangedDamage:15,range:27 },
  blood_awaken: { id:"blood_awaken",name:"혁린약동•재",line:"blood",mana:180,cooldown:900,warningTicks: 1200,strengthAmp:2,resistanceAmp:2,hungerAmp:2 },
  blood_bind: { id:"blood_bind",name:"적박 (궁극기)",line:"blood",mana:250,cooldown:1300,duration:100,damage:10,healCap:12,radius:16,ultimate:true },
  // ---------- 해커: v0.4.37 기능 복원 ----------
  hacker_hack: { id:"hacker_hack",name:"해킹",line:"hacker",mana:40,cooldown:120,range:32 },
  hacker_dummy_data: { id:"hacker_dummy_data",name:"더미데이터",line:"hacker",mana:30,cooldown:40,range:20,damage: 26,speed:1.65,hitRadius:0.9,lifetime:50 },
  hacker_reboot: { id:"hacker_reboot",name:"시스템 재부팅",line:"hacker",mana:150,cooldown:800,radius:30 },
  hacker_stealth: { id:"hacker_stealth",name:"은신",line:"hacker",mana:60,cooldown:400 },
  hacker_copy: { id:"hacker_copy",name:"카피",line:"hacker",mana:200,cooldown:1200 },
  hacker_ddos: { id:"hacker_ddos",name:"디도스",line:"hacker",mana:110,cooldown:240,radius:10,damage:30 },
  hacker_emp: { id:"hacker_emp",name:"EMP (궁극기)",line:"hacker",mana:350,cooldown:1800,radius:30,duration:100,ultimate:true },

  // ---------- 히든 직업 용기사 ----------
  dragon_triple_strike: { id:"dragon_triple_strike", name:"삼조격", line:"dragon", mana:70, cooldown:220, duration:200, bonusDamage:12, thirdStun:20 },
  dragon_gaze: { id:"dragon_gaze", name:"용의 시선", line:"dragon", mana:85, cooldown:180, damage:20, range:28, speed:1.5, markDuration:200, markMultiplier:1.3 },
  dragon_charge: { id:"dragon_charge", name:"무쌍돌격", line:"dragon", mana:110, cooldown:320, damage:18, range:26, speed:1.75 },
  // 5초 강제 질주: 21블록/초 = 1.05블록/틱. 충돌한 서로 다른 적에 대해 14 × 1.5^(n-1).
  dragon_sprint: { id:"dragon_sprint", name:"용의 질주", line:"dragon", mana:110, cooldown:400, duration:100, speed:21, damage:14, multiplier:1.5, hitRadius:1.35 },
  dragon_domain: { id:"dragon_domain", name:"현월수룡 (궁극기)", line:"dragon", mana:320, cooldown:1200, radius:7, duration:200, entryDamage:20, pulseDamage:8, allyBuffTicks:60, ultimate:true },
  dragon_breath: { id:"dragon_breath", name:"용의 브레스", line:"dragon", mana:120, cooldown:280, damage:24, range:28, speed:1.45, hitRadius:1.3, burnSeconds:4 },

  // ---------- 검 계열 ----------
  // 검사 계열 공통 스킬: 작골염양. 미지정 수치는 임시 밸런스(틱 단위).
  sword_flame_spin: {
    id: "sword_flame_spin", name: "작골염양", line: "sword", damageByJob: { sword_saint: 18.9 },
    mana: 50, cooldown: 160, damage: 12.6,
    duration: 28, hits: 3, hitInterval: 10, radius: 3.4, burnSeconds: 4,
    dashRange: 7, dashSpeed: 1.0,
  },
  // 검성·타락검성 공용: 세로 방향 호 참격을 3회 연속 발동. 수치는 임시 밸런스.
  sword_kenho_rando: {
    id: "sword_kenho_rando", name: "검호 난도", line: "sword",
    mana: 65, cooldown: 220, damagePerSlash: 12,
    slashes: 3, slashInterval: 12, reach: 5.5, halfWidth: 1.9,
  },
  // 검성·타락검성 공용: 시전 전방 180도로 확장되는 큰 초승달·작은 달 참격.
  sword_moon_dragon: {
    id: "sword_moon_dragon", name: "월룡륜미", line: "sword",
    mana: 70, cooldown: 180, damage: 28,
    range: 12, radius:12, duration:12, height:3, knockback: 0.8,
  },
  sword_spin: {
    id: "sword_spin", name: "360도 휘두르기", line: "sword", damageByJob: { sword_saint: 20 },
    mana: 20, cooldown: 30,
    damage: 10, radius: 4.5, knockback: 1.3,
  },
  sword_wave: {
    id: "sword_wave", name: "검기 방출", line: "sword", damageByJob: { sword_saint: 21 },
    mana: 30, cooldown: 60,
    damage: 14, range: 18, speed: 1.0, width: 2.5, hitRadius: 2.6, knockback: 1.0,
  },
  sword_annihilation: {
    id: "sword_annihilation", name: "섬멸", line: "sword",
    // 발도를 대체하는 검성 전용 광역 참격. 전방 15블록 순간이동, 이동 경로 반경 4블록, 대상별 1회 피해.
    mana: 40, cooldown: 100,
    damage: 52, range: 15, hitRadius: 4, knockback: 1.15,
  },
  sword_rush: {
    id: "sword_rush", name: "러쉬 (궁극기)", line: "sword", damageByJob: { sword_saint: 6 },
    mana: 80, cooldown: 300,
    damage: 4, radius: 10, hits: 8, stepTicks: 4, // 40% of original 10 damage per hit
  },
  fallen_sword_frenzy: {
    id: "fallen_sword_frenzy", name: "광분화", line: "sword",
    mana: 100, cooldown: 1200, // 60초 쿨타임
    duration: 400, lifesteal: 0.30, // 20초간 모든 직접 피해의 30% 흡혈
  },

  // ---------- 히든 흑마법사 / 반물질의 신 ----------
  // 피해량·마나·쿨타임은 추후 밸런싱하기 쉽도록 이곳에서 수정합니다.
  dark_energy: {
    id: "dark_energy", name: "암흑 에너지", line: "necro",
    mana: 55, cooldown: 80, damage: 12, range: 28, speed: 1.35, radius: 1.5, burstRadius: 3, burstDamage: 12,
  },
  event_horizon: {
    id: "event_horizon", name: "사건의 지평선", line: "necro",
    mana: 120, cooldown: 360, size: 5, duration: 100, stunTicks: 60, range: 22,
  },
  dark_chains: {
    id: "dark_chains", name: "암흑사슬", line: "necro",
    mana: 140, cooldown: 360, delay: 60, damage: 36, range: 35,
  },
  dark_touch: {
    id: "dark_touch", name: "어둠의 손길", line: "necro",
    mana: 85, cooldown: 200, damage: 18, range: 35, pullDistance: 2.25,
  },
  // '암흑 에너지' 스킬은 흑마법사=투사체, 반물질의 신=지름 6 범위 공격으로 나뉩니다.
  // 범위형은 아래 burstRadius/burstDamage로 독립 밸런싱할 수 있습니다.
  death_domain: {
    id: "death_domain", name: "죽음의 영역", line: "necro",
    mana: 250, cooldown: 900, duration: 240, radius: 5, range: 32,
    darknessLevel: 0, weaknessLevel: 1, // 나약함 II = amplifier 1
  },

  // ---------- 히든 망나니 / 야수 ----------
  // 패시브: 엔티티 처치 시 5초간 신속 III + 재생 III (망나니/야수 공통)
  berserker_charge: {
    id: "berserker_charge", name: "돌진", line: "berserker",
    mana: 70, cooldown: 160, damage: 14, range: 17, wallHitDistance: 2.5,
    wallStunTicks: 40, wallDamageMultiplier: 2,
  },
  berserker_slam: {
    id: "berserker_slam", name: "내려찍기", line: "berserker",
    mana: 80, cooldown: 240, radius: 5, riseBlocks: 10,
    damage: 18,
  },
  berserker_execute: {
    id: "berserker_execute", name: "참수", line: "berserker",
    mana: 90, cooldown: 360, nextMeleeMultiplier: 4, executeThreshold: 0.15,
  },
  beast_berserk: {
    id: "beast_berserk", name: "광분화", line: "berserker",
    mana: 180, cooldown: 1000, duration: 400,
    hungerAmp: 2, strengthAmp: 3, resistanceAmp: 1, speedAmp: 1,
    beastLifesteal: 0.15, beastAbsorptionAmp: 3, // 야수 전용 추가 피해 흡수 IV (20초)
  },
  berserker_triple_slash: {
    id: "berserker_triple_slash", name: "3연베기", line: "berserker",
    mana: 70, cooldown: 200, damagePerHit: 12,
    slashes: 3, slashInterval: 10, stepDistance: 1.1, range: 3.5, halfWidth: 2.2,
  },
  beast_flame_burst: {
    id: "beast_flame_burst", name: "화염방사", line: "berserker",
    mana: 110, cooldown: 240, shots: 3, shotInterval: 7,
    damagePerShot: 15, range: 26, speed: 1.6, hitRadius: 1.3, burnSeconds: 3,
  },
  beast_grand_explosion: {
    id: "beast_grand_explosion", name: "대폭발 (궁극기)", line: "berserker",
    mana: 280, cooldown: 1200, delay: 20, radius: 15, damage: 70, ultimate: true,
  },

  // ---------- 궁수 계열 ----------
  // 활은 화살을 사용해 평타 사격을 할 수 있습니다. 스킬은 활을 당겼다 놓으면 추가 발동합니다.
  archer_airwalk: {
    id: "archer_airwalk", name: "에어워크", line: "archer",
    mana: 45, cooldown: 300, // 견습 3초 / 궁수 8초 / 천궁 15초
  },
  archer_rapid: {
    id: "archer_rapid", name: "연사", line: "archer",
    mana: 35, cooldown: 60, // 견습 2발 / 궁수 3발 / 천궁 5발
  },
  archer_fire: {
    id: "archer_fire", name: "불 화살", line: "archer", maxTier: 2,
    mana: 50, cooldown: 100, burnSeconds: 5,
  },
  archer_ice: {
    id: "archer_ice", name: "얼음 화살", line: "archer",
    mana: 60, cooldown: 110, slowAmp: 2, extraDamage: 3,
  },
  archer_lightning: {
    id: "archer_lightning", name: "전기 화살", line: "archer",
    mana: 65, cooldown: 120, stunTicks: 20,
  },
  archer_homing: {
    id: "archer_homing", name: "유도 화살 활성화", line: "archer",
    mana: 160, cooldown: 2600, duration: 2400,
  },
  archer_bomb: {
    id: "archer_bomb", name: "폭탄 화살", line: "archer",
    mana: 90, cooldown: 140, extraDamage: 14,
  },
  archer_triple: {
    id: "archer_triple", name: "삼연사", line: "archer",
    mana: 45, cooldown: 80, damage: 7, range: 30, spread: 0.15,
  },
  archer_pierce: {
    id: "archer_pierce", name: "관통 화살", line: "archer",
    mana: 65, cooldown: 140, damage: 18, range: 38, hitRadius: 1.3,
  },
  archer_judgement: {
    id: "archer_judgement", name: "심판", line: "archer",
    // 내가 가장 최근 피해를 준 살아 있는 대상에게 대형 빛의 화살 발사 (65피해, 35초, 300MP).
    mana: 300, cooldown: 700, damage: 65, range: 96, speed: 2.6,
  },
  archer_heavenly: {
    id: "archer_heavenly", name: "천공의 일격 (궁극기)", line: "archer",
    mana: 260, cooldown: 600, damage: 42, range: 52, hitRadius: 2.3, burstDamage: 18, burstRadius: 5,
  },

  // ---------- 무투가 계열 ----------
  // 모든 피해에는 힘 버프 증가분이 더해지고, 강펀치가 켜져 있으면 그 합계가 2배가 됩니다.
  fist_strong: {
    id: "fist_strong", name: "강펀치", line: "fist",
    mana: 30, cooldown: 200,
    hits: 3, duration: 600, // 다음 3번 공격 2배 (30초 안에 못 쓰면 사라짐)
  },
  fist_flurry: {
    id: "fist_flurry", name: "난타", line: "fist",
    mana: 80, cooldown: 300,
    damage: 20, hits: 10, stepTicks: 2, radius: 10, range: 8, returnToOrigin: false, // damage 는 전체 합계 (체력 10칸)
  },
  fist_smash: {
    id: "fist_smash", name: "스매쉬", line: "fist",
    mana: 60, cooldown: 200, // 쿨타임 10초
    damage: 28, range: 32, charge: 20, dashSpeed: 3.5, coneDot: 0.85, knockback: 1.6,
  },

  // ---------- 무투가 계열: 권골 ----------
  fist_wind: {
    id: "fist_wind", name: "풍압", line: "fist",
    mana: 90, cooldown: 260, // 마법 피해, 쿨타임 13초
    damage: 20, damageType: "magic", range: 12, coneDot: 0.5, knockback: 6.4, vertical: 1.8,
  },
  fist_face: {
    id: "fist_face", name: "안면강타", line: "fist",
    mana: 100, cooldown: 400,
    radius: 6, stun: 20, damage: 0, // 6블럭 안 모두 끌어오고 1초 기절
  },
  fist_stance: {
    id: "fist_stance", name: "강자의 자세", line: "fist",
    mana: 100, cooldown: 600,
    freeze: 20, buffTicks: 300, strengthAmp: 1, resistanceAmp: 1, // 1초 자기 둔화, 15초간 힘 2 / 저항 2
  },
  fist_finale: {
    id: "fist_finale", name: "대미장식 (궁극기)", line: "fist",
    mana: 300, cooldown: 1200,
    damage: 60, radius: 12, rise: 15, riseTicks: 20, hoverTicks: 10, holdOffset: 2.0, slamSpeed: 3, // 체력 30칸 고정 피해
  },

  // ---------- 마법 계열: 견습 ----------
  mage_fireball: {
    id: "mage_fireball", name: "파이어볼", line: "mage", maxTier: 2,
    mana: 15, cooldown: 20,
    damage: 6, speed: 1.2, range: 40, radius: 3, hitRadius: 1.2,
  },
  mage_lightning: {
    id: "mage_lightning", name: "라이트닝", line: "mage", maxTier: 2,
    mana: 40, cooldown: 60,
    damage: 8, range: 24,
  },

  // ---------- 마법 계열: 2차 ----------
  mage_flame_on: {
    id: "mage_flame_on", name: "플레임온", line: "mage",
    mana: 150, cooldown: 240,
    damage: 10, radius: 15, burn: 5,
  },
  mage_thunder: {
    id: "mage_thunder", name: "번개", line: "mage",
    mana: 200, cooldown: 300,
    damage: 25, strikes: 3, interval: 8, range: 20, // damage 는 3번 합계
  },
  mage_ice_fire: {
    id: "mage_ice_fire", name: "아이스불", line: "mage",
    mana: 180, cooldown: 280,
    damage: 12, radius: 10, slowAmp: 5, duration: 80,
  },
  mage_gravity: {
    id: "mage_gravity", name: "그래비티", line: "mage",
    mana: 160, cooldown: 280,
    damage: 10, radius: 10, slowAmp: 3, duration: 100, // 구속 4단계 = amplifier 3
  },
  mage_bombaye: {
    id: "mage_bombaye", name: "봄바예", line: "mage",
    mana: 120, cooldown: 120,
    damage: 14, shots: 4, damageMultiplier: 0.3,
    speed: 0.9, range: 30, seekRadius: 25, turn: 0.25, hitRadius: 1.4,
  },

  // ---------- 마법 계열: 3차 ----------
  sage_meteor: {
    id: "sage_meteor", name: "메테오", line: "mage",
    mana: 700, cooldown: 400,
    damage: 58, radius: 8, delay: 40, range: 40,
    homingRange: 60, // 마지막으로 맞춘 대상이 이 거리 안에 있으면 그 대상에게 낙하
  },
  sage_raiyoung: {
    id: "sage_raiyoung", name: "뇌영", line: "mage",
    mana: 450, cooldown: 100,
    damage: 28.8, speed: 2.2, range: 35, hitRadius: 2.0, burst: 4, // 36의 0.8배
  },
  sage_zero: {
    id: "sage_zero", name: "절대영도", line: "mage",
    mana: 1000, cooldown: 450,
    damage: 38.4, radius: 20, freeze: 100, // 48의 0.8배
  },
  sage_blackhole: {
    id: "sage_blackhole", name: "블랙홀", line: "mage",
    mana: 900, cooldown: 500,
    damage: 50, radius: 12, duration: 120, range: 30, homingRange: 60, // damage 는 지속 시간 동안의 합계
  },
  sage_sphere: {
    id: "sage_sphere", name: "초구체", line: "mage",
    mana: 750, cooldown: 250,
    damage: 50, speed: 1.6, range: 32, hitRadius: 2.2, radius: 7, // 기본 피해 50
  },
  sage_zoltraak: {
    id: "sage_zoltraak", name: "졸트라크", line: "mage",
    mana: 1000, cooldown: 600,
    damage: 96, range: 50, width: 5, // 120의 0.8배
  },

  // ---------- 성직자 계열 ----------
  // trigger: "hit" 이면 우클릭이 아니라 '좌클릭으로 엔티티를 때릴 때' 발동합니다.
  // 맞은 엔티티를 즉시 치유. 회복량은 직업별 하트 칸 수(견습 2 / 성직자 4 / 천사 5 / 미카엘 8칸)
  holy_heal: {
    id: "holy_heal", name: "힐", line: "holy", trigger: "hit",
    mana: 30, cooldown: 20,
    healByJob: { apprentice: 2, cleric: 4, angel: 5, michael: 8 }, // 하트 칸 수
  },
  // 저항/힘 단계는 표시 단계(1=I, 2=II, 3=III) 기준입니다. 성직자 1 / 천사 2 / 미카엘 3
  holy_shield: {
    id: "holy_shield", name: "불굴의 방패", line: "holy",
    mana: 60, cooldown: 400,
    absorption: 4, duration: 6000, radius: 8, // absorption 은 효과 단계(0=1단계)
    resistanceByJob: { cleric: 1, angel: 2, michael: 3 },
  },
  holy_sword: {
    id: "holy_sword", name: "불굴의 검", line: "holy",
    mana: 70, cooldown: 400,
    duration: 6000, radius: 8,
    strengthByJob: { cleric: 1, angel: 2, michael: 3 },
  },
  // 좌클릭으로 엔티티 1명에게 부착. 성직자와 maxDistance 칸 넘게 떨어지면 사라집니다.
  // 붙어 있는 동안 성직자는 허기, 대상은 재생/신속/흡수 (표시 단계, 전직할수록 +1)
  holy_orb: {
    id: "holy_orb", name: "축복의 구슬", line: "holy", trigger: "hit",
    mana: 90, cooldown: 160,
    maxDistance: 30,
    hunger: 2, // 성직자에게 걸리는 허기 단계(표시 단계)
    regenByJob: { cleric: 2, angel: 3, michael: 4 },
    speedByJob: { cleric: 2, angel: 3, michael: 4 },
    absorbByJob: { cleric: 8, angel: 9, michael: 10 },
  },
  holy_pray: {
    id: "holy_pray", name: "기도", line: "holy",
    mana: 0, cooldown: 24000, // 게임 시간 하루(20분)에 1번
    duration: 200, // 기도 중 실명/구속 시간(틱)
  },
  angel_bless: {
    id: "angel_bless", name: "천사의 축복", line: "holy",
    mana: 250, cooldown: 1200,
    heal: 20, regen: 2, resistance: 1, speed: 1, duration: 6000, radius: 12,
  },
  angel_zone: {
    id: "angel_zone", name: "축복 영역", line: "holy",
    mana: 300, cooldown: 3000,
    heal: 2, radius: 8, duration: 300, slowAmp: 1,
    regen: 4,
  },

  // ---------- 성직자 계열: 타락천사 ----------
  // 좌클릭으로 적을 때리면 발동: 상대에게 하트 8칸(damage 16) 피해 (다른 효과 없음)
  fallen_sacrifice: {
    id: "fallen_sacrifice", name: "Sacrifice", line: "holy", trigger: "hit",
    mana: 50, cooldown: 100,
    damage: 16, // 하트 8칸
  },
  fallen_sword: {
    id: "fallen_sword", name: "파멸의 검", line: "holy",
    mana: 120, cooldown: 200,
    damage: 38, range: 18, width: 1.8, wither: 1, witherTicks: 100,
  },
  fallen_shield: {
    id: "fallen_shield", name: "저주의 방패", line: "holy",
    mana: 100, cooldown: 400,
    resistance: 2, duration: 100, reflect: 1.0, witherTicks: 60, // 5초간 받은 피해의 100% 반사
  },
  fallen_curse: {
    id: "fallen_curse", name: "타천의 저주", line: "holy",
    mana: 130, cooldown: 240,
    damage: 10, range: 30, duration: 200,
  },
  fallen_orb: {
    id: "fallen_orb", name: "저주의 구슬", line: "holy",
    mana: 150, cooldown: 200,
    damage: 16, speed: 1.1, range: 35, hitRadius: 1.4, radius: 5, duration: 200,
    maxDistance: 30, vulnerability: 0.35, witherAmp: 2,
  },
  fallen_zone: {
    id: "fallen_zone", name: "저주 영역", line: "holy",
    mana: 300, cooldown: 800,
    damage: 3, radius: 9, duration: 300,
  },
  fallen_end: {
    id: "fallen_end", name: "종말 (궁극기)", line: "holy",
    mana: 800, cooldown: 1200,
    damage: 60, radius: 25, charge: 80, chargeSlowAmp: 0, // charge 동안 종말이 타천을 따라다니며 내부 적을 사슬로 묶음(구속 1)
  },
};

// Use the real player job, including when a borrowed skill supplies another job context.
export function damageForJob(def,jobId){return def.damageByJob?.[jobId]??def.damage;}
```

## RPG_BP_v0578/scripts/main.js

```javascript
import {registerTheWorld} from './skills/theWorld.js';
import {registerTheWorldStand} from './core/theWorldStand.js';
import {registerTpa} from './core/tpa.js';
import {registerPvpArena} from './core/pvpArena.js';
import {restoreFullHealth} from './core/healthRecovery.js';
import {registerPeaceZones} from './core/peaceZones.js';
import {registerXpShop} from './core/xpShop.js';
import {registerStatusTablet} from './core/statusTablet.js';
import {registerHuntingZones} from './core/huntingZones.js';
import {registerPeaceServer} from './core/peaceServer.js';
import {registerPvp} from './core/pvp.js';
import {registerRarityLabels} from './core/rarityLabels.js';
import {registerBossMotion} from './core/bossMotion.js';
import {registerGptNpc} from './core/gptNpc.js';
import {registerTemples} from './core/temples.js';
import {registerProgressionItems} from './core/progressionItems.js';
import {registerHolyRecovery} from './core/holyRecovery.js';
import {registerAngelHealing} from './core/angelHealing.js';
import {registerCombatContributions} from './core/combatContribution.js';
import {registerLifesteal} from './core/lifesteal.js';
import {registerWarGod} from './skills/warGod.js';
import {registerDungeons} from './core/dungeons.js';
import { world, system } from "@minecraft/server";
import { CONFIG } from "./config.js";
import { JOBS, findJob } from "./data/jobs.js";
import { SKILLS } from "./data/skills.js";
import {
  getJobId, setJobId, setSkills, getSkills, getMana, setMana, maxManaOf, getProgress,
  clearPlayerRuntime, getDisabled,
} from "./core/playerData.js";
import {
  requestAdvance, ensureWeapon, resetToApprentice, giveStarter, requirementStatus,
} from "./core/advance.js";
import { registerInput, handleFriendlyHit } from "./core/input.js";
import { registerTracking } from "./core/tracking.js";
import { registerHoly } from "./skills/holy.js";
import { registerFist } from "./skills/fist.js";
import { registerFistCoating } from "./core/fistCoating.js";
import { registerArcher } from "./skills/archer.js";
import { registerNecromancer } from "./skills/necro.js";
import { registerDarkMagic } from "./skills/dark.js";
import { registerSandbag } from "./core/sandbag.js";
import { registerWings } from "./core/wings.js";
import { registerFrenzy } from "./core/frenzy.js";
import { registerBerserker } from "./skills/berserker.js";
import { registerBloodMagic } from "./skills/blood.js";
import { registerHacker } from "./skills/hacker.js";
import { registerHarpoon } from "./skills/harpoon.js";
import { registerFrost } from "./skills/frost.js";
import { registerFire } from "./skills/fire.js";
import { registerAssassin } from "./skills/assassin.js";
import { registerTechnomancer } from "./skills/technomancer.js";
import { registerDemon } from "./skills/demon.js";
import { registerCelestial } from "./skills/celestial.js";
import { registerHito } from "./core/hito.js";
import { registerSwordFlameSpin } from "./skills/swordFlameSpin.js";
import { registerDragon } from "./skills/dragon.js";

import { registerSocial } from "./core/social.js";
import { registerFriendlyFire } from "./core/friendlyFire.js";
import { registerGolemBoss } from "./core/golemBoss.js";
import { registerCyclopsBoss } from "./core/cyclopsBoss.js";
import { runRpgAction } from "./core/rpgActions.js";
import { setCommandHandler, setGameplayStatus } from "./core/commandBridge.js";
import { startHud } from "./core/hud.js";
import { registerRpgStats, applyVitality } from "./core/stats.js";
import { registerBeginnerGuide, recordBeginnerKill } from './core/beginner.js';
import { getEquipmentBonuses, registerEquipment } from './core/equipment.js';
import { registerRuneDrops } from './core/runeDrops.js';
import {registerMonsterLevels} from './core/monsterLevels.js';
import { registerJobQuests } from './core/jobQuests.js';
import { registerNewTrials } from './core/newTrials.js';

// 처음 들어온 플레이어는 견습으로 시작
function initPlayer(player) {
  if (getJobId(player)) return;
  setJobId(player, "apprentice");
  setSkills(player, []);
  setMana(player, maxManaOf(player));
}

// Start gameplay from the same static ES module graph as command registration.
// Each feature is isolated so a single faulty handler does not disable commands.
let gameplayInitialized = false;
export function initializeGameplay() {
  if (gameplayInitialized) return { state: "ready", errors: [] };
  gameplayInitialized = true;
  const errors = [];
  const safe = (name, fn) => {
    try { fn(); }
    catch (error) {
      const info = name + ": " + String(error?.stack ?? error);
      errors.push(info);
      console.warn("[rpg/gameplay] " + info);
    }
  };

  // Link the command bridge even if a nonessential skill system fails.
  setCommandHandler(runRpgAction);
  safe('hunting zones',registerHuntingZones);
  safe('monster levels',registerMonsterLevels);
  safe('peace server',registerPeaceServer);
  safe('peace zones',registerPeaceZones);
  safe('The World',registerTheWorld);
  safe('The World stand',registerTheWorldStand);
  safe('TPA',registerTpa);
  safe('PvP arena',registerPvpArena);
  safe('PvP and PK',registerPvp);
  safe('XP shop and potion',registerXpShop);
  safe('status tablet',()=>registerStatusTablet(runRpgAction));
  safe('celestial temple',registerTemples);
  safe('GPT guide NPC',registerGptNpc);
  safe('item rarity labels',registerRarityLabels);
  safe('boss motions',registerBossMotion);
  safe("playerSpawn", () => world.afterEvents.playerSpawn.subscribe((ev) => {
    const p = ev.player;
    system.runTimeout(() => {
      try {
        initPlayer(p);
        ensureWeapon(p);
        applyVitality(p);
        if(!ev.initialSpawn){restoreFullHealth(p);system.runTimeout(()=>{try{restoreFullHealth(p);}catch{}},2);}
      } catch { /* player unavailable */ }
    }, 20);
  }));
  safe("playerLeave", () => world.afterEvents.playerLeave.subscribe((ev) => clearPlayerRuntime(ev.playerId)));
  safe("manaRegen", () => system.runInterval(() => {
    for (const p of world.getAllPlayers()) {
      try {
        initPlayer(p);
        const max = maxManaOf(p);
        const mana = getMana(p);
        if (mana < max) {const gear=getEquipmentBonuses(p);setMana(p,mana+Math.max(1,max*CONFIG.mana.regenRate)+gear.manaRegen+max*gear.manaRegenPercent);}
        ensureWeapon(p);
      } catch { /* player not ready */ }
    }
  }, 20));

  const features = [
    ["rune lifesteal",()=>registerLifesteal()],
    ["war god",()=>registerWarGod()],
    ["combat contribution", () => registerCombatContributions()],
    ["holy recovery", () => registerHolyRecovery()],
    ["angel healing rewards", () => registerAngelHealing()],
    ["progression items", () => registerProgressionItems()],
    ["RPG stats", () => registerRpgStats(recordBeginnerKill)],
    ["beginner guide", () => registerBeginnerGuide()],
    ['RPG equipment', () => registerEquipment()],
    ['vanilla rune drops', () => registerRuneDrops()],
    ['job quests', () => registerJobQuests()],
    ['new job trials', () => registerNewTrials()],
    ['party dungeons', () => registerDungeons()],
    ["social", () => registerSocial()],
    ["friendly fire", () => registerFriendlyFire(handleFriendlyHit)],
    ["golem boss", () => registerGolemBoss()],
    ["cyclops boss", () => registerCyclopsBoss()],
    ["input", () => registerInput()],
    ["작골염양 점프 감지", () => registerSwordFlameSpin()],
    ["tracking", () => registerTracking()],
    ["holy", () => registerHoly()],
    ["fist", () => registerFist()],
    ["fist coating", () => registerFistCoating()],
    ["archer", () => registerArcher()],
    ["necromancer", () => registerNecromancer()],
    ["dark magic", () => registerDarkMagic()],
    ["sandbag", () => registerSandbag()],
    ["wings", () => registerWings()],
    ["frenzy", () => registerFrenzy()],
    ["berserker", () => registerBerserker()],
    ["blood magic", () => registerBloodMagic()],
    ["hacker", () => registerHacker()],
    ["harpoon", () => registerHarpoon()],
    ["frost", () => registerFrost()],
    ["fire", () => registerFire()],
    ["assassin", () => registerAssassin()],
    ["technomancer", () => registerTechnomancer()],
    ["demon", () => registerDemon()],
    ["celestial", () => registerCelestial()],
    ["dragon knight", () => registerDragon()],
    ["job NPC / hito", () => registerHito()],
    ["HUD", () => startHud()],
  ];
  for (const [name, start] of features) safe(name, start);

  safe("script events", () => system.afterEvents.scriptEventReceive.subscribe((ev) => {
    if (!ev.id.startsWith("rpg:") || ["rpg:ping", "rpg:diag"].includes(ev.id)) return;
    const p = ev.sourceEntity;
    if (!p || p.typeId !== "minecraft:player") return;
    try { runRpgAction(ev.id.slice(4), p, ev.message); }
    catch (error) { console.warn("[rpg/script-event] " + String(error)); }
  }));
  setGameplayStatus(errors.length ? "partial" : "ready", errors);
  console.warn("[rpg/gameplay] initialized; errors=" + errors.length);
  return { state: errors.length ? "partial" : "ready", errors };
}
```

## RPG_BP_v0578/scripts/skills/theWorld.js

```javascript
import {duelOf} from '../core/pvp.js';
import {world,system,InputPermissionCategory,EntityDamageCause} from '@minecraft/server';
import {attack as martialAttack} from './warGod.js';
import {SKILLS} from '../data/skills.js';
import {getJobId} from '../core/playerData.js';
import {getTargets,flatDir,isSolidAt,knockback,heldItemId,hurt} from '../core/util.js';
import {areAllies} from '../core/social.js';
import {isBoss} from '../core/crowdControl.js';
import {scriptedHit,setCombatHealth,noteManualDamage} from '../core/combatContext.js';
import {registerStoppedTarget,takeStoppedDamage,isTimeFrozen} from '../core/timeStopDamage.js';
export {isTimeFrozen};
const MARK='rpg:the_world_freeze_v1',TAG='rpg_the_world_frozen',WEAK='rpg:the_world_native_guard',HIGH='rpg:the_world_high_until';
const sessions=new Map(),frames=new Map(),basics=new Map();let registered=false;
const valid=e=>{try{return Boolean(e?.isValid);}catch{return false;}};
const living=e=>valid(e)&&(e.getComponent('minecraft:health')?.currentValue??0)>0;
const effectData=(e,name)=>{const f=e.getEffect(name);return f?{duration:f.duration,amplifier:f.amplifier}:null;};
function enemy(p,e){return valid(e)&&p.id!==e.id&&e.typeId!=='rpg:the_world_stand'&&!areAllies(p,e)&&!e.typeId.startsWith('rpg:hito_')&&e.typeId!=='rpg:gpt_guide'&&(e.typeId!=='minecraft:player'||!['creative','spectator'].includes(String(e.getGameMode?.()).toLowerCase()));}
export function restoreFrozenEntity(e){
 const raw=e.getDynamicProperty(MARK);if(raw===undefined){try{e.removeTag(TAG);}catch{}return true;}
 let saved;try{saved=JSON.parse(raw);}catch{console.warn('[The World] corrupt recovery marker');return false;}
 let okay=true;const elapsed=Math.max(0,Math.floor((Date.now()-saved.at)/50));
 for(const name of ['slowness','weakness'])try{e.removeEffect(name);const old=saved.effects[name];if(old&&old.duration>elapsed)e.addEffect(name,old.duration-elapsed,{amplifier:old.amplifier,showParticles:false});}catch{okay=false;}
 if(e.typeId==='minecraft:player')for(const [category,enabled]of saved.permissions??[])try{e.inputPermissions.setPermissionCategory(category,enabled);}catch{okay=false;}
 try{e.clearVelocity();const projectile=e.getComponent('minecraft:projectile');if(projectile)projectile.shoot(saved.velocity);else e.applyImpulse(saved.velocity);}catch{okay=false;}
 if(okay){e.setDynamicProperty(MARK,undefined);e.removeTag(TAG);}return okay;
}
function sharedGuild(a,b){try{return a?.typeId==='minecraft:player'&&b?.typeId==='minecraft:player'&&a.getTags().some(t=>t.startsWith('rpg_guild_')&&b.hasTag(t));}catch{return false;}}
function freeze(p,e,session){
 if(frames.has(e.id)||!enemy(p,e)||sharedGuild(p,e))return false;
 const projectile=e.getComponent('minecraft:projectile');
 if(projectile?.owner&&(projectile.owner.id===p.id||areAllies(p,projectile.owner)||sharedGuild(p,projectile.owner)))return false;
 if(!projectile&&!living(e))return false;
 if(e.getDynamicProperty(MARK)!==undefined&&!restoreFrozenEntity(e))return false;
 const saved={at:Date.now(),effects:{slowness:effectData(e,'slowness'),weakness:effectData(e,'weakness')},velocity:e.getVelocity(),permissions:[]};
 if(e.typeId==='minecraft:player')try{for(const category of [...new Set([InputPermissionCategory.Movement,InputPermissionCategory.Camera,...Object.values(InputPermissionCategory).filter(v=>typeof v==='number')])])saved.permissions.push([category,e.inputPermissions.isPermissionCategoryEnabled(category)]);}catch{saved.permissions=[];}
 e.setDynamicProperty(MARK,JSON.stringify(saved));e.addTag(TAG);e.clearVelocity();
 let mode='position';
 if(saved.permissions.length>=2)try{for(const category of [InputPermissionCategory.Movement,InputPermissionCategory.Camera])e.inputPermissions.setPermissionCategory(category,false);mode='inputPermissions';}catch{mode='position';}
 if(!projectile){e.addEffect('slowness',isBoss(e)?50:100,{amplifier:255,showParticles:false});e.addEffect('weakness',isBoss(e)?50:100,{amplifier:255,showParticles:false});}
 frames.set(e.id,{entity:e,owner:p,location:{...e.location},rotation:e.getRotation(),dimension:e.dimension.id,until:system.currentTick+(isBoss(e)?50:100),duel:duelOf(p),mode});session.ids.add(e.id);registerStoppedTarget(e,p);return true;
}
function releaseFrame(id,applyDamage=true){
 const f=frames.get(id);frames.delete(id);const deferred=takeStoppedDamage(id);if(!f)return;
 try{if(valid(f.entity))restoreFrozenEntity(f.entity);}catch(error){console.warn('[The World restore] '+error);}
 if(!applyDamage||!deferred?.damage||!living(f.entity)||!living(f.owner)||areAllies(f.owner,f.entity)||(f.duel&&(f.duel.finished||duelOf(f.owner)!==f.duel||duelOf(f.entity)!==f.duel)))return;
 const target=f.entity,source=f.owner,amount=deferred.damage;
 scriptedHit(target,source,()=>{try{if(target.applyDamage(amount,{cause:EntityDamageCause.override,damagingEntity:source})!==false)return true;}catch{}
  const hp=target.getComponent('minecraft:health');if(!hp||hp.currentValue<=0)return false;const before=hp.currentValue;setCombatHealth(target,source,hp,Math.max(0,before-amount));noteManualDamage(source,target,Math.min(before,amount));return true;});
}
export function stopTheWorld(p,applyDamage=true){const session=sessions.get(p.id);if(!session)return false;sessions.delete(p.id);for(const id of session.ids)releaseFrame(id,applyDamage);return true;}
export function startTheWorld(p){
 if(sessions.has(p.id))return false;
 const session={ids:new Set(),dimension:p.dimension.id,until:system.currentTick+100};sessions.set(p.id,session);
 try{for(const e of p.dimension.getEntities({location:p.location,maxDistance:20})){if(session.ids.size>=60)break;try{freeze(p,e,session);}catch(error){try{restoreFrozenEntity(e);}catch{}console.warn('[The World freeze] '+error);}}}
 catch(error){stopTheWorld(p,false);throw error;}
 p.sendMessage('§6더 월드! §7시간이 멈췄습니다.');return true;
}
export function tickTheWorld(){
 for(const [id,s]of sessions){const p=world.getAllPlayers().find(p=>p.id===id);if(!living(p)||p.dimension.id!==s.dimension||system.currentTick>=s.until||getJobId(p)!=='the_world'){stopTheWorld(p??{id},living(p)&&p.dimension.id===s.dimension&&getJobId(p)==='the_world');continue;}}
 for(const [id,f]of frames){
  if(!valid(f.entity)||!living(f.owner)||f.entity.dimension.id!==f.dimension||system.currentTick>=f.until||areAllies(f.owner,f.entity)){releaseFrame(id);continue;}
  try{f.entity.teleport(f.location,{dimension:f.entity.dimension,rotation:f.rotation,keepVelocity:false});f.entity.clearVelocity();}catch{releaseFrame(id);}
 }
}
export function recoverStoppedEntities(){
 for(const name of ['overworld','nether','the_end'])try{for(const e of world.getDimension(name).getEntities({tags:[TAG]}))if(!frames.has(e.id))try{restoreFrozenEntity(e);}catch(error){console.warn('[The World recovery] '+error);}}catch{}
 for(const p of world.getAllPlayers())if(!frames.has(p.id)&&p.getDynamicProperty(MARK)!==undefined)try{restoreFrozenEntity(p);}catch{}
}
function saveNativeGuard(p){if(p.getDynamicProperty(WEAK)===undefined)p.setDynamicProperty(WEAK,JSON.stringify({at:Date.now(),effect:effectData(p,'weakness')}));p.addEffect('weakness',40,{amplifier:255,showParticles:false});}
function restoreNativeGuard(p){const raw=p.getDynamicProperty(WEAK);if(raw===undefined)return;const saved=JSON.parse(raw);p.removeEffect('weakness');const ticks=(Date.now()-saved.at)/50;if(saved.effect?.duration>ticks)p.addEffect('weakness',Math.ceil(saved.effect.duration-ticks),{amplifier:saved.effect.amplifier,showParticles:false});p.setDynamicProperty(WEAK,undefined);}
export function tickWorldGuards(){for(const p of world.getAllPlayers())try{if(isTimeFrozen(p))continue;if(getJobId(p)==='the_world'&&(heldItemId(p)==='rpg:the_world_arrow'||sessions.has(p.id))&&!isTimeFrozen(p))saveNativeGuard(p);else restoreNativeGuard(p);}catch(error){console.warn('[The World melee guard] '+error);}}
function punch(p,job,d){return martialAttack(p,job,d);}
function finisher(p,job,d){return martialAttack(p,job,d);}
function roller(p,job,d){const dir=flatDir(p),origin=p.location;let at=origin;for(let n=.5;n<=3;n+=.5){const q={x:origin.x+dir.x*n,y:origin.y,z:origin.z+dir.z*n};if(isSolidAt(p.dimension,q)||isSolidAt(p.dimension,{...q,y:q.y+1}))break;at=q;}p.teleport(at,{dimension:p.dimension,keepVelocity:false});return martialAttack(p,job,d);}
function high(p){p.setDynamicProperty(HIGH,system.currentTick+160);p.addEffect('speed',160,{amplifier:1,showParticles:false});return true;}
export const RUNNERS={world_muda:punch,world_finish:finisher,world_roller:roller,world_high:high,world_stop:startTheWorld};
export function registerTheWorld(){
 if(registered)return;registered=true;
 world.beforeEvents.itemUse.subscribe(e=>{if(sessions.has(e.source.id)&&e.itemStack?.typeId!=='rpg:the_world_arrow')e.cancel=true;});
 system.runInterval(tickTheWorld,1);system.runInterval(tickWorldGuards,1);system.runInterval(recoverStoppedEntities,100);system.run(recoverStoppedEntities);
 world.afterEvents.entitySpawn.subscribe(ev=>system.run(()=>{try{const owner=ev.entity.getComponent('minecraft:projectile')?.owner;if(owner&&isTimeFrozen(owner))ev.entity.remove();}catch{}}));
 world.afterEvents.entityLoad.subscribe(ev=>{if(!frames.has(ev.entity.id))try{restoreFrozenEntity(ev.entity);}catch{}});
 world.afterEvents.playerSpawn.subscribe(ev=>system.run(()=>{try{if(!frames.has(ev.player.id))restoreFrozenEntity(ev.player);restoreNativeGuard(ev.player);}catch{}}));
 world.afterEvents.playerDimensionChange.subscribe(ev=>stopTheWorld(ev.player,false));
 world.afterEvents.playerLeave.subscribe(ev=>{stopTheWorld({id:ev.playerId},false);basics.delete(ev.playerId);});
 world.afterEvents.entityDie.subscribe(ev=>{stopTheWorld(ev.deadEntity,false);if(frames.has(ev.deadEntity.id))releaseFrame(ev.deadEntity.id,false);});
 world.afterEvents.entityHitEntity.subscribe(ev=>{const p=ev.damagingEntity,t=ev.hitEntity;if(p?.typeId!=='minecraft:player'||getJobId(p)!=='the_world'||heldItemId(p)!=='rpg:the_world_arrow'||isTimeFrozen(p)||!enemy(p,t))return;
  const gap=Number(p.getDynamicProperty(HIGH)??0)>system.currentTick?5:10;if((basics.get(p.id)??-100)>system.currentTick-gap)return;basics.set(p.id,system.currentTick);hurt(t,p,8,'physical',{ignoreInvulnerability:true});});
}
```

## RPG_BP_v0578/scripts/skills/warGod.js

```javascript
import {deferStoppedDamage} from '../core/timeStopDamage.js';
import {setCombatHealth} from '../core/combatContext.js';
import {applyStatus,ccTicks,ccScale} from '../core/crowdControl.js';
import {warGodAttackFx,warGodImpactFx,warGodFinaleFx} from './warGodFx.js';
import {world,system,EntityDamageCause} from '@minecraft/server';
import {getTargets,flatDir,isSolidAt,knockback,particle,sound,heldItemId} from '../core/util.js';
import {skillDamageFor} from '../core/stats.js';
import {scriptedHit,currentScriptHit,rememberScriptKiller,noteManualDamage,setBasicHitHandler,BLACK_FLASH_KEY} from '../core/combatContext.js';
import {healLifesteal} from '../core/lifesteal.js';
import {areAllies} from '../core/social.js';
import {CONFIG} from '../config.js';
import {notify} from '../core/hud.js';
export {BLACK_FLASH_KEY};
function valid(p,t){return t&&t.id!==p.id&&!areAllies(p,t)&&!['minecraft:armor_stand','minecraft:item','minecraft:xp_orb'].includes(t.typeId)&&(!t.typeId.startsWith('rpg:hito_'))&&(t.typeId!=='minecraft:player'||(CONFIG.pvp&&!['creative','spectator'].includes(String(t.getGameMode?.()).toLowerCase())));}
function cone(p,def){
 const origin=p.location,dir=flatDir(p);return getTargets(p,{x:origin.x,y:origin.y+1,z:origin.z},def.range+1,.9).filter(t=>{
  if(!valid(p,t))return false;const dx=t.location.x-origin.x,dz=t.location.z-origin.z,along=dx*dir.x+dz*dir.z,side=Math.abs(dx*dir.z-dz*dir.x);
  if(along<0||along>def.range||side>def.width||Math.abs(t.location.y-origin.y)>3)return false;
  for(let step=.5;step<Math.hypot(dx,dz);step+=.5){const distance=Math.hypot(dx,dz)||1;if(isSolidAt(p.dimension,{x:origin.x+dx/distance*step,y:origin.y+1,z:origin.z+dz/distance*step}))return false;}
  return true;
 }).slice(0,8);
}
function hit(p,t,damage){
 return scriptedHit(t,p,()=>{
  const amount=skillDamageFor(p,damage,'physical');
  if(deferStoppedDamage(t,p,amount))return true;
  try{if(t.applyDamage(amount,{cause:EntityDamageCause.override,damagingEntity:p})!==false)return true;}catch{}
  const hp=t.getComponent('minecraft:health');if(!hp||hp.currentValue<=0)return false;
  if(amount>=hp.currentValue)rememberScriptKiller(p,t);const before=hp.currentValue;setCombatHealth(t,p,hp,Math.max(0,before-amount));noteManualDamage(p,t,Math.min(before,amount));return true;
 });
}
function impact(p,t,kick){
 const at={x:t.location.x,y:t.location.y+1,z:t.location.z};particle(p.dimension,'minecraft:critical_hit_emitter',at);warGodImpactFx(p,t);sound(p.dimension,'mob.irongolem.hit',at,{volume:.6,pitch:kick ? 0.7 : 1.1});
}
export function attack(p,j,d){
 const targets=cone(p,d);if(!targets.length){notify(p,'§7전방에 공격할 적이 없습니다.');return false;}
 const boosted=p.getDynamicProperty(BLACK_FLASH_KEY)===true;let mult=1;if(boosted){p.setDynamicProperty(BLACK_FLASH_KEY,undefined);mult=3;notify(p,'§4천멸격! §f스킬 피해 ×3');sound(p.dimension,'mob.wither.spawn',p.location,{volume:.4,pitch:1.6});}
 const originDimension=p.dimension.id;let i=0;
 const pulse=()=>{
  if(!p.isValid||p.dimension.id!==originDimension||!(p.getComponent('minecraft:health')?.currentValue>0))return;
  warGodAttackFx(p,d,i);
  for(const t of cone(p,d)){if(hit(p,t,d.damagePerHit*mult)){impact(p,t,d.kick);if(i===d.hits-1){const direction=flatDir(p);knockback(t,direction.x,direction.z,d.kick?2.1:1.1,.2);try{applyStatus(t,'slowness',d.kick?20:10,{amplifier:1,showParticles:false});}catch{}}}}
  i++;
 };
 pulse();for(let n=1;n<d.hits;n++)system.runTimeout(pulse,n*d.interval);return true;
}
export function armBlackFlash(p){
 if(p.getDynamicProperty(BLACK_FLASH_KEY)===true){notify(p,'§7이미 천멸격이 준비되어 있습니다.');return false;}
 p.setDynamicProperty(BLACK_FLASH_KEY,true);notify(p,'§c천멸격 준비 · 다음 스킬 ×3');sound(p.dimension,'mob.wither.spawn',p.location,{volume:.5,pitch:1.8});return true;
}
export function applyBlackFlash(){}
export const RUNNERS={war_god_punch:attack,war_god_flurry:attack,war_god_kick:attack,war_god_kick_flurry:attack,war_god_black_flash:armBlackFlash};
let registered=false;export function registerWarGod(){if(registered)return;registered=true;}
```

## RPG_Commands_BP_v0578/manifest.json

```json
{
  "format_version": 2,
  "header": {
    "name": "RPG Commands BP v0.5.89",
    "description": "RPG v0.5.89 · 더 월드·시간 정지·스탠드·1000골드 TPA",
    "uuid": "533cff27-4591-48cb-85fb-540c8bcbcc72",
    "version": [
      0,
      5,
      89
    ],
    "min_engine_version": [
      1,
      21,
      100
    ]
  },
  "modules": [
    {
      "type": "script",
      "language": "javascript",
      "uuid": "9518dff9-cc69-4ea4-adb6-2762b6291c45",
      "entry": "scripts/commands.js",
      "version": [
        0,
        5,
        89
      ]
    }
  ],
  "dependencies": [
    {
      "uuid": "a1b49558-2a78-4754-bae7-1c5ec7a84303",
      "version": [
        0,
        5,
        89
      ]
    },
    {
      "module_name": "@minecraft/server",
      "version": "2.2.0"
    }
  ]
}
```

## RPG_Commands_BP_v0578/scripts/commands.js

```javascript
// RPG command registry, isolated in a separate behavior pack.
// Even if complex RPG combat/skill imports fail, /rpg:ping can still register.
import * as mc from "@minecraft/server";
const VERSION = "0.5.78";
const commands = [
  ['tpa','플레이어 이동 요청 · 이동 성공 시 1000골드'],
  ['tpaaccept','받은 플레이어 이동 요청 수락·거절'],
  ['pvpcreate','관리자: 1000 250 1000에 PvP장 생성',null,true],
  ['pkclear','관리자: 플레이어 PK 패널티 해제','player',true],
  ['peace','관리자: 현재 위치 반경 200 평화구역 설정·해제','mode',true],
  ['pvp','결투 신청·수락·항복·PK 상태','player'],
  ['gptnpc','관리자: GPT RPG 질문 NPC 생성',null,true],
  ['temple','천체 신전 위치·활성 제단·후예 전직'],
  ['templecreate','관리자: 앞쪽 빈 지역에 천체 신전 생성',null,true],
  ['templeremove','관리자: 신전 등록 해제·건물 보존',null,true],
  ['fuse','RPG 장비 등급 합성'],
  ['transfer','RPG 갑옷 강화 추출·이전'],
  ['ask','이 애드온의 직업·스킬·제작·던전 정보 질문','question'],
  ['dungeoncreate','관리자: 난이도 1~10 또는 all 동굴 생성·누락 복구','difficulty',true],
  ['dungeonremove','관리자: 던전 등록 해제 (건축물 보존)','name',true],
  ['dungeon','난이도 목록·파티장 입장 시 파티원 자동 등록','difficulty'],
  ['dungeonstart','대기실 인원 1~4명 출발'],
  ['dungeonsolo','해당 난이도 혼자 바로 시작','difficulty'],
  ['dungeonleave','대기 취소·사망 후 귀환 (전투 중 불가)'],
  ['dungeonloot','보관된 던전 보급품 받기'],
  ['skillguide','모든 직업·스킬 상세 도감'],
  ['runes','룬 전용 보관함·장착'],
  ['jobs','직업별 전직 조건·시련'],
  ['craft', '갑옷·룬 제작법과 획득 도감'],
  ['upgrade', '갑옷·룬·공용 무기 강화'],
  ['wallet', '골드·강화석·룬 가루 지갑'],
  ['equipment', 'RPG 갑옷·룬 보관·장착'],
  ['guide', '모험 시작·견습 5계열 자유 체험'],
  ["rulebook", "RPG 서버 종합 룰북"],
  ["ping", "명령어 등록 진단"],
  ["diag", "명령어 및 스크립트 진단"],
  ["menu", "카멜레온 스킬 메뉴"],
  ["job", "전직", "job"],
  ["help", "명령어 도움말"],
  ["status", "캐릭터 상태"],
  ["stats", "레벨·스탯 분배"],
  ["admin", "운영자 설정 도구", null, true],
  ["setlevel", "관리자: 레벨 설정", "level", true],
  ["setmana", "관리자: 최대 마나 설정", "amount", true],
  ["skills", "스킬 켜기·끄기"],
  ["skillbook", "스킬 도감"],
  ["learn", "스킬 습득", "skill"],
  ["swap", "카멜레온 무기 전환"],
  ["offer", "검과 책을 바칩니다"],
  ["bless", "미카엘 전직: 축복의 빛 192개 공물"],
  ["holywater", "성수 제작"],
  ["force", "관리자: 강제 전직", "job", true],
  ["reset", "관리자: 초기화", null, true],
  ["starter", "관리자: 견습 장비", "line", true],
  ["charge", "관리자: 마나·쿨타임 회복", null, true],
  ["items", "관리자: 전용 아이템", null, true],
  ["egg", "관리자: 몬스터 스폰알", null, true],
  ["hito", "관리자: 전직관 소환알 지급", "line", true],
  ["wings", "날개 켜기·끄기"],
  ["wingcheck", "날개 상태"],
  ["wingrefresh", "날개 새로고침"],
  ["guild", "길드 메뉴"],
  ["party", "파티 메뉴"],
  ["invites", "초대 확인"],
  ["boss", "관리자: 바위 골렘", null, true],
  ["cyclops", "관리자: 외눈 거인", null, true],
  ["hydra", "관리자: 히드라", null, true],
  ["techgolem", "관리자: 마공학 골렘", null, true],
  ["fistcheck", "주먹 강화 상태"]
];
const registered = new Set();
let startupDone = false;
let failures = [];
function isPlayer(p) { return !!p && p.typeId === "minecraft:player"; }
function tell(p, msg) { try { p.sendMessage(msg); } catch (e) { console.warn("[RPG commands] message: " + e); } }
function handle(action, p, arg="") {
  if (!isPlayer(p)) return;
  if (action === "ping") {
    tell(p, "§a[RPG v"+VERSION+"] 명령어 스크립트 실행 정상. 등록 "+registered.size+"/"+commands.length);
    return;
  }
  if (action === "diag") {
    tell(p,"§e[RPG v"+VERSION+"] 명령어 팩 실행 정상 ("+registered.size+"/"+commands.length+"). RPG 본체 확인: /rpg:status");
    if(failures.length) failures.slice(0,4).forEach(x=>tell(p,"§c"+x));
    tell(p,"§7게임플레이 오류는 설정 → 제작자 → 콘텐츠 로그에서 확인 가능. /function rpg_hydra 로 히드라 소환 검사 가능");
    return;
  }
  // The RPG gameplay BP independently handles this script event. It is run as
  // the player so event.sourceEntity is the correct player object.
  const message = String(arg??"").replace(/[\r\n]/g," ").trim();
  try {
    p.runCommand("scriptevent rpg:"+action+(message?" "+message:""));
  } catch (e) {
    tell(p,"§c[RPG] 명령어 전달 실패: "+String(e));
  }
}
mc.system.beforeEvents.startup.subscribe(ev => {
  startupDone = true;
  const registry = ev.customCommandRegistry;
  if(!registry){ console.error("[RPG commands] customCommandRegistry unavailable"); return; }
  const levels = mc.CommandPermissionLevel ?? mc.CustomCommandPermissionLevel;
  const params = mc.CustomCommandParamType;
  const statuses = mc.CustomCommandStatus;
  if(!levels || !params || !statuses){ console.error("[RPG commands] Script API command enums unavailable"); return; }
  for(const [name, description, arg, admin] of commands){
    try{
      const def={name:"rpg:"+name,description,permissionLevel: admin?levels.GameDirectors:levels.Any,cheatsRequired:false};
      if(arg){
        const t = name === "setlevel" ? params.Integer : params.String;
        if(name === "hito" || name === "dungeon" || name === "ask" || name === "pvp" || name === "peace" || name === "pkclear") def.optionalParameters=[{name:arg,type:t}];
        else def.mandatoryParameters=[{name:arg,type:t}];
      }
      registry.registerCommand(def,(origin,...args)=>{
        const p=origin.sourceEntity??origin.initiator;
        if(!isPlayer(p)) return {status:statuses.Failure,message:"플레이어 전용 명령어"};
        const raw=Array.isArray(args[0])?args[0][0]:args[0];
        mc.system.run(()=>handle(name,p,raw==null?"":String(raw)));
        return {status:statuses.Success};
      });
      registered.add(name);
    }catch(e){
      const msg="/rpg:"+name+" 등록 실패: "+String(e);
      failures.push(msg);console.error("[RPG commands] "+msg);
    }
  }
  try{registry.registerCommand({name:'pvp',description:'결투 신청',permissionLevel:levels.Any,cheatsRequired:false,optionalParameters:[{name:'player',type:params.String}]},(origin,arg)=>{const p=origin.sourceEntity??origin.initiator;if(!isPlayer(p))return {status:statuses.Failure};mc.system.run(()=>handle('pvp',p,arg??''));return {status:statuses.Success};});}catch{ /* Namespace-free aliases are unsupported on some Bedrock versions. */ }
  try{registry.registerCommand({name:'peace',description:'관리자: 반경 200 평화구역',permissionLevel:levels.GameDirectors,cheatsRequired:false,optionalParameters:[{name:'mode',type:params.String}]},(origin,arg)=>{const p=origin.sourceEntity??origin.initiator;if(!isPlayer(p))return {status:statuses.Failure};mc.system.run(()=>handle('peace',p,arg??''));return {status:statuses.Success};});}catch{console.warn('[RPG] /peace 별칭 미지원: /rpg:peace 사용');}
  try{registry.registerCommand({name:'pvpcreate',description:'관리자: PvP장 생성',permissionLevel:levels.GameDirectors,cheatsRequired:false},origin=>{const p=origin.sourceEntity??origin.initiator;if(!isPlayer(p))return {status:statuses.Failure};mc.system.run(()=>handle('pvpcreate',p,''));return {status:statuses.Success};});}catch{console.warn('[RPG] /pvpcreate 별칭 미지원: /rpg:pvpcreate 사용');}
  console.warn("[RPG v"+VERSION+"] registered "+registered.size+"/"+commands.length+" commands");
});
// Allow commands even if a particular client doesn't display slash commands.
try {
  const chat=mc.world.beforeEvents?.chatSend;
  if(chat?.subscribe) chat.subscribe(ev=>{
    const msg=String(ev.message??"").trim();
    if(!/^!(?:rpg|rulebook|hito|pvp)(?:\s|$)/i.test(msg))return;
    ev.cancel=true;
    let action="help",arg="";
    if(/^!pvp(?:\s|$)/i.test(msg)){action='pvp';arg=msg.replace(/^!pvp\s*/i,'');}
    else if(/^!rulebook$/i.test(msg))action="rulebook";
    else if(/^!hito(?:\s|$)/i.test(msg)){action="hito";arg=msg.replace(/^!hito\s*/i,"");}
    else {const parts=msg.replace(/^!rpg\s*/i,"").trim().split(/\s+/);action=(parts.shift()||"help").toLowerCase();arg=parts.join(" ");}
    const p=ev.sender;
    mc.system.run(()=>handle(action,p,arg));
  });
}catch(e){console.warn("[RPG commands] chat fallback disabled: "+e);}
// Direct /scriptevent rpg:ping and rpg:diag allow diagnosing even if custom
// command registration is unsupported on the current Bedrock version.
try {
  mc.system.afterEvents.scriptEventReceive.subscribe(ev=>{
    if(ev.id!=="rpg:ping"&&ev.id!=="rpg:diag") return;
    if(isPlayer(ev.sourceEntity))mc.system.run(()=>handle(ev.id.slice(4),ev.sourceEntity));
  });
}catch(e){console.warn("[RPG commands] event diagnostic fallback disabled: "+e);}
```

## RPG_RP_v0578/animations/the_world_hidden.animation.json

```json
{
  "format_version": "1.8.0",
  "animations": {
    "animation.rpg.the_world.hide": {
      "loop": true,
      "bones": {
        "root": {
          "scale": 0
        }
      }
    }
  }
}
```

## RPG_RP_v0578/attachables/the_world_arrow.json

```json
{
  "format_version": "1.10.0",
  "minecraft:attachable": {
    "description": {
      "identifier": "rpg:the_world_arrow",
      "materials": {
        "default": "entity_alphatest"
      },
      "textures": {
        "default": "textures/items/the_world_arrow"
      },
      "geometry": {
        "default": "geometry.rpg.the_world_hidden"
      },
      "animations": {
        "hide": "animation.rpg.the_world.hide"
      },
      "scripts": {
        "animate": [
          "hide"
        ]
      },
      "render_controllers": [
        "controller.render.item_default"
      ]
    }
  }
}
```

## RPG_RP_v0578/entity/the_world_stand.entity.json

```json
{
  "format_version": "1.10.0",
  "minecraft:client_entity": {
    "description": {
      "identifier": "rpg:the_world_stand",
      "materials": {
        "default": "entity_alphatest"
      },
      "textures": {
        "default": "textures/entity/the_world_stand"
      },
      "geometry": {
        "default": "geometry.rpg.the_world_stand"
      },
      "render_controllers": [
        "controller.render.default"
      ]
    }
  }
}
```

## RPG_RP_v0578/manifest.json

```json
{
  "format_version": 2,
  "header": {
    "name": "RPG Addon RP v0.5.89",
    "description": "RPG v0.5.89 · 더 월드·시간 정지·스탠드·1000골드 TPA",
    "uuid": "8fa45523-e936-4258-a195-01e5e62e814c",
    "version": [
      0,
      5,
      89
    ],
    "min_engine_version": [
      1,
      21,
      100
    ]
  },
  "modules": [
    {
      "type": "resources",
      "uuid": "7ede5e36-9a5f-44d1-9aa0-9dc38822fa99",
      "version": [
        0,
        5,
        89
      ]
    }
  ]
}
```

## RPG_RP_v0578/models/entity/the_world_hidden.geo.json

```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.rpg.the_world_hidden",
        "texture_width": 64,
        "texture_height": 64,
        "visible_bounds_width": 1,
        "visible_bounds_height": 1
      },
      "bones": [
        {
          "name": "root",
          "pivot": [
            0,
            0,
            0
          ],
          "cubes": [
            {
              "origin": [
                0,
                0,
                0
              ],
              "size": [
                1,
                1,
                1
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## RPG_RP_v0578/models/entity/the_world_stand.geo.json

```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.rpg.the_world_stand",
        "texture_width": 64,
        "texture_height": 64,
        "visible_bounds_width": 3,
        "visible_bounds_height": 3,
        "visible_bounds_offset": [
          0,
          1.5,
          0
        ]
      },
      "bones": [
        {
          "name": "body",
          "pivot": [
            0,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                12,
                -2
              ],
              "size": [
                8,
                12,
                4
              ],
              "uv": [
                16,
                16
              ]
            }
          ]
        },
        {
          "name": "head",
          "parent": "body",
          "pivot": [
            0,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                24,
                -4
              ],
              "size": [
                8,
                8,
                8
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        },
        {
          "name": "rightArm",
          "pivot": [
            -5,
            22,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8,
                12,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                40,
                16
              ]
            }
          ]
        },
        {
          "name": "leftArm",
          "pivot": [
            5,
            22,
            0
          ],
          "cubes": [
            {
              "origin": [
                4,
                12,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                32,
                48
              ]
            }
          ]
        },
        {
          "name": "rightLeg",
          "pivot": [
            -2,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                0,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                0,
                16
              ]
            }
          ]
        },
        {
          "name": "leftLeg",
          "pivot": [
            2,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                0,
                0,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                16,
                48
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## RPG_RP_v0578/textures/entity/the_world_stand.png

바이너리 PNG. ZIP에 원본 전체 포함.

## RPG_RP_v0578/textures/item_texture.json

```json
{
  "resource_pack_name": "rpg",
  "texture_name": "atlas.items",
  "texture_data": {
    "rpg_apprentice_sword": {
      "textures": "textures/items/native_grips/apprentice_sword"
    },
    "rpg_swordsman_sword": {
      "textures": "textures/items/native_grips/swordsman_sword"
    },
    "rpg_sword_saint_sword": {
      "textures": "textures/items/native_grips/sword_saint_sword"
    },
    "rpg_fallen_sword_saint_sword": {
      "textures": "textures/items/native_grips/fallen_sword_saint_sword"
    },
    "rpg_apprentice_book": {
      "textures": "textures/items/apprentice_book"
    },
    "rpg_mage_book": {
      "textures": "textures/items/mage_book"
    },
    "rpg_sage_staff": {
      "textures": "textures/items/native_grips/sage_staff"
    },
    "rpg_fallen_angel_wings": {
      "textures": "textures/items/fallen_angel_wings"
    },
    "rpg_michael_wings": {
      "textures": "textures/items/michael_wings"
    },
    "rpg_angel_hands": {
      "textures": "textures/items/angel_hands"
    },
    "rpg_cleric_cross": {
      "textures": "textures/items/cleric_cross"
    },
    "rpg_apprentice_cross": {
      "textures": "textures/items/apprentice_cross"
    },
    "rpg_blessing_light": {
      "textures": "textures/items/blessing_light"
    },
    "rpg_holy_water": {
      "textures": "textures/items/holy_water"
    },
    "rpg_apprentice_fist": {
      "textures": "textures/items/apprentice_fist"
    },
    "rpg_fighter_fist": {
      "textures": "textures/items/fighter_fist"
    },
    "rpg_fist_bone_fist": {
      "textures": "textures/items/fist_bone_fist"
    },
    "rpg_angel_wing_toggle": {
      "textures": "textures/items/angel_wing_toggle"
    },
    "rpg_michael_wing_toggle": {
      "textures": "textures/items/michael_wing_toggle"
    },
    "rpg_fallen_wing_toggle": {
      "textures": "textures/items/fallen_wing_toggle"
    },
    "rpg_apprentice_bow": {
      "textures": "textures/items/apprentice_bow"
    },
    "rpg_archer_bow": {
      "textures": "textures/items/archer_bow"
    },
    "rpg_heavenly_bow": {
      "textures": "textures/items/heavenly_bow"
    },
    "rpg_sandbag_spawner": {
      "textures": "textures/blocks/rpg_sandbag"
    },
    "rpg_apprentice_tome": {
      "textures": "textures/items/apprentice_tome"
    },
    "rpg_necromancer_staff": {
      "textures": "textures/items/necromancer_staff"
    },
    "rpg_deathlord_staff": {
      "textures": "textures/items/deathlord_staff"
    },
    "rpg_mangnani_axe": {
      "textures": "textures/items/native_grips/mangnani_axe"
    },
    "rpg_beast_axe": {
      "textures": "textures/items/native_grips/beast_axe"
    },
    "rpg_necromancer_contract": {
      "textures": "textures/items/necromancer_contract"
    },
    "rpg_mangnani_contract": {
      "textures": "textures/items/mangnani_contract"
    },
    "rpg_hacker_core": {
      "textures": "textures/items/hacker_core"
    },
    "rpg_blood_crystal": {
      "textures": "textures/items/blood_crystal"
    },
    "rpg_assassin_blade": {
      "textures": "textures/items/native_grips/assassin_blade"
    },
    "rpg_shadow_assassin_blade": {
      "textures": "textures/items/native_grips/shadow_assassin_blade"
    },
    "rpg_blood_spear": {
      "textures": "textures/items/blood_spear"
    },
    "rpg_harpooner_harpoon": {
      "textures": "textures/items/native_grips/harpooner_harpoon"
    },
    "rpg_revenant_harpoon": {
      "textures": "textures/items/native_grips/revenant_harpoon"
    },
    "rpg_frost_gauntlet": {
      "textures": "textures/items/frost_gauntlet"
    },
    "rpg_fire_gauntlet": {
      "textures": "textures/items/fire_gauntlet"
    },
    "rpg_tech_staff": {
      "textures": "textures/items/native_grips/tech_staff"
    },
    "rpg_tech_aux_arm": {
      "textures": "textures/items/tech_aux_arm"
    },
    "rpg_demon_scythe": {
      "textures": "textures/items/native_grips/demon_scythe"
    },
    "rpg_sun_heir_axe": {
      "textures": "textures/items/native_grips/sun_heir_axe"
    },
    "rpg_moon_heir_blade": {
      "textures": "textures/items/moon_heir_blade"
    },
    "rpg_moon_heir_focus": {
      "textures": "textures/items/moon_heir_focus"
    },
    "rpg_chameleon_flame_blade": {
      "textures": "textures/items/native_grips/chameleon_flame_blade"
    },
    "rpg_chameleon_moon_scythe": {
      "textures": "textures/items/native_grips/chameleon_moon_scythe"
    },
    "rpg_admin_settings": {
      "textures": "textures/items/admin_settings"
    },
    "rpg_dragon_knight_lance": {
      "textures": "textures/items/native_grips/dragon_knight_lance"
    },
    "rpg_adventurer_helmet": {
      "textures": "textures/items/rpg_equipment/adventurer_helmet"
    },
    "rpg_adventurer_chestplate": {
      "textures": "textures/items/rpg_equipment/adventurer_chestplate"
    },
    "rpg_adventurer_leggings": {
      "textures": "textures/items/rpg_equipment/adventurer_leggings"
    },
    "rpg_adventurer_boots": {
      "textures": "textures/items/rpg_equipment/adventurer_boots"
    },
    "rpg_warden_helmet": {
      "textures": "textures/items/rpg_equipment/warden_helmet"
    },
    "rpg_warden_chestplate": {
      "textures": "textures/items/rpg_equipment/warden_chestplate"
    },
    "rpg_warden_leggings": {
      "textures": "textures/items/rpg_equipment/warden_leggings"
    },
    "rpg_warden_boots": {
      "textures": "textures/items/rpg_equipment/warden_boots"
    },
    "rpg_astral_helmet": {
      "textures": "textures/items/rpg_equipment/astral_helmet"
    },
    "rpg_astral_chestplate": {
      "textures": "textures/items/rpg_equipment/astral_chestplate"
    },
    "rpg_astral_leggings": {
      "textures": "textures/items/rpg_equipment/astral_leggings"
    },
    "rpg_astral_boots": {
      "textures": "textures/items/rpg_equipment/astral_boots"
    },
    "rpg_rune_power": {
      "textures": "textures/items/rpg_equipment/rune_power"
    },
    "rpg_rune_ward": {
      "textures": "textures/items/rpg_equipment/rune_ward"
    },
    "rpg_rune_mana": {
      "textures": "textures/items/rpg_equipment/rune_mana"
    },
    "rpg_rune_arcane": {
      "textures": "textures/items/rpg_equipment/rune_arcane"
    },
    "rpg_rune_vitality": {
      "textures": "textures/items/rpg_equipment/rune_vitality"
    },
    "rpg_enhancement_catalyst": {
      "textures": "textures/items/rpg_equipment/enhancement_catalyst"
    },
    "rpg_random_skillbook": {
      "textures": "textures/items/rpg_equipment/random_skillbook"
    },
    "rpg_rune_absorption": {
      "textures": "textures/items/rpg_equipment/rune_absorption"
    },
    "rpg_war_god_gauntlets": {
      "textures": "textures/items/rpg_equipment/war_god_gauntlets"
    },
    "rpg_stat_reset_scroll": {
      "textures": "textures/items/rpg_equipment/stat_reset_scroll"
    },
    "rpg_job_waiver": {
      "textures": "textures/items/rpg_equipment/job_waiver"
    },
    "rpg_abyssal_helmet": {
      "textures": "textures/items/rpg_equipment/warden_helmet"
    },
    "rpg_abyssal_chestplate": {
      "textures": "textures/items/rpg_equipment/warden_chestplate"
    },
    "rpg_abyssal_leggings": {
      "textures": "textures/items/rpg_equipment/warden_leggings"
    },
    "rpg_abyssal_boots": {
      "textures": "textures/items/rpg_equipment/warden_boots"
    },
    "rpg_armor_transfer_seal": {
      "textures": "textures/items/rpg_equipment/random_skillbook"
    },
    "rpg_redmoon_blade": {
      "textures": "textures/items/native_grips/redmoon_blade"
    },
    "rpg_magitech_sword": {
      "textures": "textures/items/native_grips/magitech_sword"
    },
    "rpg_magitech_greatsword": {
      "textures": "textures/items/native_grips/magitech_greatsword"
    },
    "rpg_elite_knight_greatsword": {
      "textures": "textures/items/native_grips/elite_knight_greatsword"
    },
    "rpg_frost_greatsword": {
      "textures": "textures/items/native_grips/frost_greatsword"
    },
    "rpg_flame_greatsword": {
      "textures": "textures/items/native_grips/flame_greatsword"
    },
    "rpg_status_tablet": {
      "textures": "textures/items/status_tablet"
    },
    "rpg_xp_boost_potion": {
      "textures": "textures/items/holy_water"
    },
    "rpg_the_world_arrow": {
      "textures": "textures/items/the_world_arrow"
    }
  }
}
```

## RPG_RP_v0578/textures/items/the_world_arrow.png

바이너리 PNG. ZIP에 원본 전체 포함.

## RPG_RP_v0578/ui/server_form.json

```json
{
  "namespace": "server_form",
  "main_screen_content": {
    "type": "panel",
    "size": [
      "100%",
      "100%"
    ],
    "controls": [
      {
        "server_form_factory": {
          "type": "factory",
          "control_ids": {
            "long_form": "@server_form.rpg_form_router",
            "custom_form": "@server_form.custom_form"
          }
        }
      }
    ]
  },
  "rpg_form_router": {
    "type": "panel",
    "size": [
      "100%",
      "100%"
    ],
    "controls": [
      {
        "vanilla@server_form.long_form": {
          "bindings": [
            {
              "binding_name": "#title_text"
            },
            {
              "binding_type": "view",
              "source_property_name": "(not (#title_text = '§bSTATUS §8| §f상태창'))",
              "target_property_name": "#visible"
            }
          ]
        }
      },
      {
        "status@server_form.rpg_status_window": {
          "bindings": [
            {
              "binding_name": "#title_text"
            },
            {
              "binding_type": "view",
              "source_property_name": "(#title_text = '§bSTATUS §8| §f상태창')",
              "target_property_name": "#visible"
            }
          ]
        }
      }
    ]
  },
  "rpg_status_window": {
    "type": "panel",
    "size": [
      "96%",
      "94%"
    ],
    "max_size": [
      620,
      330
    ],
    "layer": 5,
    "controls": [
      {
        "background": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            "100%",
            "100%"
          ],
          "layer": 0,
          "color": [
            0.015,
            0.045,
            0.09
          ]
        }
      },
      {
        "top_edge": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            "100%",
            2
          ],
          "layer": 2,
          "offset": [
            0,
            0
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "top_middle",
          "anchor_to": "top_middle"
        }
      },
      {
        "bottom_edge": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            "100%",
            2
          ],
          "layer": 2,
          "offset": [
            0,
            0
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "bottom_middle",
          "anchor_to": "bottom_middle"
        }
      },
      {
        "left_edge": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            2,
            "100%"
          ],
          "layer": 2,
          "offset": [
            0,
            0
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "left_middle",
          "anchor_to": "left_middle"
        }
      },
      {
        "right_edge": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            2,
            "100%"
          ],
          "layer": 2,
          "offset": [
            0,
            0
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "right_middle",
          "anchor_to": "right_middle"
        }
      },
      {
        "title": {
          "type": "label",
          "text": "§b§lSTATUS §r§7// §f모험가 단말",
          "size": [
            "90%",
            14
          ],
          "anchor_from": "top_middle",
          "anchor_to": "top_middle",
          "offset": [
            0,
            10
          ],
          "layer": 3
        }
      },
      {
        "divider": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            "94%",
            1
          ],
          "layer": 2,
          "offset": [
            0,
            43
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "top_middle",
          "anchor_to": "top_middle"
        }
      },
      {
        "profile": {
          "type": "panel",
          "size": [
            "30%",
            "100% - 58px"
          ],
          "anchor_from": "top_left",
          "anchor_to": "top_left",
          "offset": [
            "3%",
            50
          ],
          "layer": 3,
          "controls": [
            {
              "emblem": {
                "type": "image",
                "texture": "textures/items/status_tablet",
                "size": [
                  "66%",
                  72
                ],
                "layer": 0,
                "anchor_from": "top_middle",
                "anchor_to": "top_middle"
              }
            },
            {
              "stats": {
                "type": "label",
                "text": "#form_text",
                "size": [
                  "100%",
                  "100% - 78px"
                ],
                "anchor_from": "top_left",
                "anchor_to": "top_left",
                "offset": [
                  0,
                  78
                ],
                "font_size": "small",
                "text_alignment": "left",
                "layer": 4,
                "bindings": [
                  {
                    "binding_name": "#form_text"
                  }
                ]
              }
            }
          ]
        }
      },
      {
        "menu_grid": {
          "type": "grid",
          "size": [
            "62%",
            "100% - 62px"
          ],
          "anchor_from": "top_right",
          "anchor_to": "top_right",
          "offset": [
            "-3%",
            52
          ],
          "grid_dimensions": [
            4,
            5
          ],
          "grid_item_template": "server_form.rpg_status_card",
          "collection_name": "form_buttons",
          "bindings": [
            {
              "binding_name": "#form_button_contents",
              "binding_name_override": "#maximum_grid_items"
            }
          ]
        }
      }
    ]
  },
  "rpg_status_card": {
    "type": "panel",
    "size": [
      "25%",
      "20%"
    ],
    "controls": [
      {
        "card@common_buttons.light_text_button": {
          "size": [
            "96%",
            "94%"
          ],
          "$pressed_button_name": "button.form_button_click",
          "$button_text": "#form_button_text",
          "$button_text_binding_type": "collection",
          "$button_text_grid_collection_name": "form_buttons",
          "$button_text_max_size": [
            "98%",
            24
          ],
          "bindings": [
            {
              "binding_type": "collection_details",
              "binding_collection_name": "form_buttons"
            }
          ]
        }
      },
      {
        "accent": {
          "type": "image",
          "texture": "textures/ui/White",
          "size": [
            "88%",
            1
          ],
          "layer": 2,
          "offset": [
            0,
            -2
          ],
          "color": [
            0.1,
            0.85,
            1
          ],
          "anchor_from": "bottom_middle",
          "anchor_to": "bottom_middle"
        }
      }
    ]
  }
}
```

