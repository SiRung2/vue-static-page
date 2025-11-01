<template>
  <!-- 하나의 풀스크린 Stage -->
  <main class="stage" role="main" aria-label="Chuseok Night" @click="onStageClick">
    <!-- 별 -->
    <div class="stars">
      <div v-for="s in stars" :key="s.id" class="star"
           :style="{ left: s.x + 'vw', top: s.y + 'svh', animationDelay: s.delay + 's', animationDuration: s.duration + 's' }"/>
    </div>

    <!-- 보름달 (우상단, 클릭: 달빛 토글) -->
    <svg class="moon" viewBox="0 0 160 160" @click.stop="toggleMoonGlow" role="img" :aria-label="moonGlow ? '달빛 강하게' : '달빛 부드럽게'">
      <defs>
        <radialGradient id="g" cx="50%" cy="42%" r="60%">
          <stop offset="0%"  :stop-color="moonGlow ? '#fff9d6' : '#fff1ae'"/>
          <stop offset="65%" :stop-color="moonGlow ? '#ffe793' : '#ffd777'"/>
          <stop offset="100%" :stop-color="moonGlow ? '#f5c85a' : '#e8b24a'"/>
        </radialGradient>
        <filter id="glow"><feGaussianBlur stdDeviation="7"/></filter>
      </defs>
      <circle cx="80" cy="80" r="56" fill="url(#g)"/>
      <circle v-if="moonGlow" cx="80" cy="80" r="56" fill="#ffd978" opacity="0.55" filter="url(#glow)"/>
      <circle cx="64" cy="62" r="6" fill="#fff8d9" opacity="0.7"/>
      <circle cx="98" cy="92" r="4" fill="#fff2c0" opacity="0.55"/>
    </svg>

    <!-- 로프 + 등불(상단 중앙 정렬) -->
    <svg class="rope" viewBox="0 0 1000 120" preserveAspectRatio="none" aria-hidden="true">
      <path d="M0,60 C250,20 750,100 1000,60" stroke="#eef2ff" stroke-width="3" fill="none"/>
    </svg>
    <div v-for="(l,i) in lanterns" :key="l.id" class="lantern"
         :style="{ left: (10 + i*(80/(lanterns.length-1))) + 'vw', top: '8svh', animationDelay: (i*0.15)+'s' }"
         role="img" :aria-label="'등불 ' + (i+1)">
      <div class="lantern-body" :style="{ background: l.color }">
        <div class="stripe"></div><div class="stripe"></div><div class="stripe"></div>
      </div>
      <div class="feet">
        <span/><span/>
      </div>
    </div>

    <!-- 떠오르는 소원 등불 (화면 어디서든 클릭하면 생성) -->
    <transition-group name="float" tag="div" class="float-layer">
      <div v-for="f in floating" :key="f.id" class="float-lantern"
           :style="{ left: f.x + 'px', top: f.y + 'px', animationDuration: f.duration + 's' }"
           role="img" aria-label="소원 등불">
        <div class="float-body"><span>{{ f.text }}</span></div>
        <div class="float-tail"></div>
      </div>
    </transition-group>

    <!-- 토끼 + 절구 (하단 중앙) -->
    <div class="rabbit-wrap" @click.stop="toggleRabbit" role="img" aria-label="옥토끼가 떡을 만들고 있어요">
      <svg class="rabbit" viewBox="0 0 420 260">
        <!-- 바닥 그림자 -->
        <ellipse cx="220" cy="216" rx="90" ry="18" fill="rgba(0,0,0,.45)"/>

        <!-- 절구 -->
        <g transform="translate(250,172)">
          <rect x="-46" y="-20" width="92" height="84" rx="24" fill="#8f5f3d"/>
          <ellipse cx="0" cy="-20" rx="48" ry="16" fill="#c89b6f"/>
          <ellipse cx="0" cy="72" rx="56" ry="14" fill="#2f2419" opacity=".6"/>
          <ellipse cx="0" cy="-26" rx="26" ry="12" fill="#fff4de"/>
        </g>

        <!-- 방아공이 -->
        <g :class="['pestle', rabbitBeats ? 'beat' : '']" transform="translate(205,56)">
          <rect x="-10" y="10" width="20" height="128" rx="10" fill="#a67b56"/>
          <rect x="-8" y="132" width="16" height="34" rx="8" fill="#a67b56"/>
        </g>

        <!-- 토끼(팔/다리/손 분리, 더 귀엽게) -->
        <g transform="translate(130,138)">
          <!-- 몸통/배 -->
          <ellipse cx="-54" cy="36" rx="54" ry="64" fill="#fff"/>
          <ellipse cx="-64" cy="50" rx="34" ry="40" fill="#fff"/>
          <!-- 머리 -->
          <ellipse cx="-98" cy="-6" rx="30" ry="28" fill="#fff"/>
          <!-- 귀 -->
          <g>
            <ellipse cx="-116" cy="-44" rx="10" ry="34" fill="#fff"/>
            <ellipse cx="-116" cy="-44" rx="5.5" ry="22" fill="#ffd5e1"/>
            <ellipse cx="-86" cy="-42" rx="10" ry="32" fill="#fff" transform="rotate(12 -86 -42)"/>
            <ellipse cx="-86" cy="-42" rx="5.5" ry="20" fill="#ffd5e1" transform="rotate(12 -86 -42)"/>
          </g>
          <!-- 얼굴 -->
          <circle cx="-110" cy="-10" r="3" fill="#333"/>
          <circle cx="-92"  cy="-9"  r="3" fill="#333"/>
          <ellipse cx="-101" cy="-2" rx="3.4" ry="2.2" fill="#f49cab"/>
          <path d="M-120,-4 h10" stroke="#808899" stroke-width="1" stroke-linecap="round"/>
          <path d="M-87,-3 h10"  stroke="#808899" stroke-width="1" stroke-linecap="round"/>

          <!-- 팔 (방아 잡는 포즈) -->
          <ellipse cx="-40" cy="10" rx="18" ry="12" fill="#fff" transform="rotate(22 -40 10)"/>
          <ellipse cx="-12" cy="0"  rx="18" ry="12" fill="#fff" transform="rotate(-10 -12 0)"/>
          <!-- 손 -->
          <circle cx="-24" cy="18" r="7" fill="#fff"/>
          <circle cx="-6"  cy="8"  r="7" fill="#fff"/>

          <!-- 다리/발 -->
          <ellipse cx="-84" cy="88" rx="20" ry="13" fill="#fff"/>
          <ellipse cx="-48" cy="90" rx="20" ry="13" fill="#fff"/>
          <!-- 꼬리 -->
          <circle cx="-22" cy="44" r="10" fill="#fff"/>
        </g>
      </svg>

      <div class="hint-mini">{{ rabbitBeats ? '토끼 방아: 동작 중 (눌러서 멈추기)' : '토끼 방아: 멈춤 (눌러서 시작)' }}</div>
    </div>

    <!-- 하단 컨트롤: 아주 간단히 -->
    <div class="controls">
      <button class="btn" @click.stop="toggleMoonGlow">{{ moonGlow ? '달빛 부드럽게' : '달빛 강하게' }}</button>
      <button class="btn outline" @click.stop="spawnLanternCenter">등불 하나 띄우기</button>
    </div>
  </main>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

type Star = { id: number; x: number; y: number; delay: number; duration: number }
type Floating = { id: number; x: number; y: number; duration: number; text: string }
type Lantern = { id: number; color: string }

const moonGlow = ref(true)
const rabbitBeats = ref(true)

function toggleMoonGlow(){ moonGlow.value = !moonGlow.value }
function toggleRabbit(){ rabbitBeats.value = !rabbitBeats.value }

/* 별: 화면 전체 분포 (vw/svh 사용 → 항상 풀스크린) */
const stars = ref<Star[]>([])
function makeStars(n = 100){
  const arr: Star[] = []
  for(let i=0;i<n;i++){
    arr.push({
      id: i,
      x: Math.random()*100,
      y: Math.random()*100,
      delay: Math.random()*3,
      duration: 2 + Math.random()*3.5
    })
  }
  stars.value = arr
}

/* 상단 등불 */
const palette = ['#ffd86b','#ff8a80','#ffd54f','#81c784','#4fc3f7','#ce93d8','#ffd86b']
const lanterns = ref<Lantern[]>(palette.map(c => ({ color: c })))

/* 소원 등불: 화면 클릭 위치에서 떠오르기 */
const floating = ref<Floating[]>([])
let seq = 0
function onStageClick(e: MouseEvent){
  const root = (e.currentTarget as HTMLElement).getBoundingClientRect()
  spawnLantern(e.clientX - root.left, e.clientY - root.top)
}
function spawnLantern(x: number, y: number){
  const duration = 8 + Math.random()*5
  const id = ++seq
  floating.value.push({ id, x, y, duration, text: '소원 성취' })
  setTimeout(()=>{ floating.value = floating.value.filter(f=>f.id!==id) }, duration*1000)
}
function spawnLanternCenter(){
  const root = document.querySelector('.stage')!.getBoundingClientRect()
  spawnLantern(root.width/2, root.height*0.75)
}

onMounted(()=>{ makeStars() })
</script>

<!-- 전역: 확실한 밤 배경 + 여백 제거 -->
<style>
:root { color-scheme: dark; }
html, body, #app { height: 100%; margin: 0; background: #000 !important; }
* { box-sizing: border-box; }
</style>

<style scoped>
/* 하나의 화면(Stage) — 단일 고정 배경 */
.stage{
  position: relative;
  width: 100vw;
  height: 100svh;            /* 모바일 주소창 변동에도 안전 */
  overflow: hidden;          /* 모든 요소를 화면 안으로 클리핑 */
  color: #e9eefc;
}
.stage::before{
  content: "";
  position: absolute; inset: 0;
  background: radial-gradient(1400px 900px at 50% -300px, #0e1a2b 0%, #0a1120 55%, #000 100%);
  z-index: -1;               /* 완전한 단일 배경: 경계선 없음 */
}

/* 별 */
.stars{ position: absolute; inset: 0; pointer-events: none; }
.star{
  position: absolute; width: 2px; height: 2px; border-radius: 50%;
  background: #fff8c6; box-shadow: 0 0 6px rgba(255,244,179,.9);
  animation: twinkle linear infinite;
}
@keyframes twinkle{ 0%,100%{ transform:scale(.9); opacity:.6 } 50%{ transform:scale(1.4); opacity:1 } }

/* 달 */
.moon{
  position: absolute; top: 8svh; right: 8vw;
  width: clamp(120px, 14vw, 200px); height: auto;
  cursor: pointer; transition: transform .25s ease;
}
.moon:hover{ transform: scale(1.03) rotate(-1deg); }

/* 로프/등불 (상단 중앙 정렬) */
.rope{
  position: absolute; top: 6svh; left: 50%; transform: translateX(-50%);
  width: 80vw; height: 12svh; pointer-events: none;
}
.lantern{
  position: absolute; width: clamp(44px, 5.2vw, 68px);
  transform-origin: top center; animation: swing 3s ease-in-out infinite;
  filter: drop-shadow(0 0 10px rgba(255,210,80,.7));
}
@keyframes swing{ 0%{transform:rotate(-3deg)} 50%{transform:rotate(3deg)} 100%{transform:rotate(-3deg)} }
.lantern-body{
  width: 100%; aspect-ratio: 1/1.2; border-radius: 24px; position: relative;
  box-shadow: inset 0 -10px 16px rgba(0,0,0,.25);
}
.stripe{ position:absolute; inset:0; border-radius:24px; border:2px solid rgba(255,255,255,.35); mix-blend-mode:soft-light }
.feet{ display:flex; gap: 6px; justify-content:center; margin-top: 4px; }
.feet span{ width: 10px; height: 12px; background:#4b5563; border-radius: 0 0 3px 3px; opacity:.9 }

/* 떠오르는 소원 등불 */
.float-layer{ position:absolute; inset:0; pointer-events:none; }
.float-lantern{
  position:absolute; display:grid; justify-items:center; gap:4px;
  animation: fly linear 1 forwards;
}
.float-body{
  min-width: 120px; max-width: 40vw; padding: 8px 12px; border-radius: 12px;
  background: #ffdf9a; color: #3b2a00; font-weight: 700; text-align:center;
  box-shadow: 0 6px 14px rgba(0,0,0,.35), inset 0 -8px 18px rgba(0,0,0,.15);
}
.float-tail{ width:18px; height:20px; background:#7a583a; border-radius:0 0 6px 6px; opacity:.9 }
@keyframes fly{
  0%  { transform: translate(-50%, -50%) translateY(0) rotate(0deg);    opacity: .0 }
  5%  { opacity: 1 }
  50% { transform: translate(-50%, -50%) translateY(-50svh) rotate(-2deg) }
  100%{ transform: translate(-50%, -50%) translateY(-100svh) rotate(2deg); opacity: .2 }
}

/* 토끼 */
.rabbit-wrap{
  position: absolute; left: 50%; bottom: 8svh; transform: translateX(-50%);
  display: grid; justify-items: center; gap: 8px; text-align: center;
}
.rabbit{ width: min(820px, 78vw); height: auto; }
.pestle{ transform-origin: 0px 140px; }
.pestle.beat{ animation: pound .9s ease-in-out infinite; }
@keyframes pound{ 0%{transform:rotate(-12deg)} 50%{transform:rotate(18deg)} 100%{transform:rotate(-12deg)} }
.hint-mini{ font-size: 12px; opacity: .75 }

/* 하단 컨트롤 (중앙) */
.controls{
  position: absolute; left: 50%; bottom: 2.8svh; transform: translateX(-50%);
  display: flex; gap: 8px; flex-wrap: wrap; justify-content: center;
}
.btn{
  appearance: none; border: 0; border-radius: 999px; padding: .6rem 1rem;
  background: #1f2937; color: #fff; font-weight: 700; cursor: pointer;
  transition: transform .12s ease, opacity .12s ease;
}
.btn:hover{ transform: translateY(-1px); opacity: .95 }
.btn:active{ transform: none; opacity: 1 }
.btn.outline{ background: transparent; border: 2px solid #8892a8 }

/* 접근성: 모션 최소화 */
@media (prefers-reduced-motion: reduce){
  .star, .lantern, .pestle.beat, .float-lantern{ animation: none !important }
}
</style>
