<template>
  <div :class="['app-shell', { projector: projectorMode, 'intro-active': introVisible }]">
    <main v-if="introVisible" class="intro-page">
      <canvas ref="introCanvas" class="intro-art" @pointermove="moveIntroArt" @pointerdown="moveIntroArt"></canvas>
      <div class="intro-topline"><span class="intro-mark">∫</span><span>ÉTS Montréal <i>·</i> MAT145</span><span class="intro-live"><b></b> Une expérience interactive</span></div>
      <div class="intro-content">
        <div class="intro-kicker">CALCUL DIFFÉRENTIEL ET INTÉGRAL</div>
        <h1>Les mathématiques<br/>prennent <em>forme.</em></h1>
        <p>Explorez les idées derrière le calcul, une courbe à la fois.</p>
        <button class="intro-enter" @click="enterPlayground">Entrer dans le playground <span>↗</span></button>
        <div class="intro-hint"><span class="hint-line"></span>Déplacez le pointeur pour explorer</div>
      </div>
      <div class="intro-equation equation-one">∫<sub>a</sub><sup>b</sup> f(x) dx</div>
      <div class="intro-equation equation-two">lim<sub>h→0</sub> &nbsp; Δy / Δx</div>
      <div class="intro-index"><span>01</span><span class="index-rule"></span><span>04</span><span>CONCEPTS À EXPLORER</span></div>
      <div class="intro-bottom"><span>Un espace pour voir, toucher et comprendre.</span><span>MAT145 STUDIO <b>©</b> 2026</span></div>
    </main>
    <header class="navbar">
      <div class="nav-inner">
        <div class="brand" @click="switchView('studio')">
          <div class="ets-logo">ÉTS</div>
          <div>
            <div class="brand-title">MAT145 Studio <span class="badge">ÉTS Montréal</span></div>
            <div class="brand-sub">Calcul différentiel et intégral • Playground interactif</div>
          </div>
        </div>

        <div class="header-note"><span class="header-note-dot"></span> Un espace pour explorer le calcul</div>

        <button :class="['btn-outline', { active: projectorMode }]" @click="projectorMode = !projectorMode">
          Mode présentation
        </button>
      </div>
    </header>

    <main class="playground-shell">
      <section class="playground-heading">
        <div>
          <div class="play-eyebrow"><span></span> PLAYGROUND MAT145</div>
          <h1>Jouez avec les idées.</h1>
          <p>Attrapez une courbe. Changez un paramètre. Voyez ce qui se passe.</p>
        </div>
        <div class="play-modes" aria-label="Choisir une idée mathématique">
          <button v-for="mode in playgroundModes" :key="mode.id" :class="['play-mode', { active: activeLab === mode.id }]" @click="selectLab(mode.id)">
            <span class="mode-symbol">{{ mode.symbol }}</span><span>{{ mode.label }}</span>
          </button>
        </div>
      </section>

      <section class="play-stage-wrap">
        <div class="stage-card playground-stage">
          <canvas ref="labCanvas" class="stage-canvas" @pointerdown="onCanvasDown" @pointermove="onCanvasMove" @pointerup="onCanvasUp" @pointercancel="onCanvasUp"></canvas>
          <div class="scene-title"><span class="scene-live"></span><div><strong>{{ currentPlay.heading }}</strong><small>{{ currentPlay.caption }}</small></div></div>
          <button class="stage-export" @click="exportPNG" aria-label="Exporter la visualisation">↗ <span>Image</span></button>
          <div class="hud-box">
            <div v-for="(line, i) in hudLines" :key="i" class="hud-line">
              <span>{{ line.label }}</span>
              <strong :style="{ color: line.color }">{{ line.value }}</strong>
            </div>
          </div>
          <div class="drag-tip">{{ activeLab === 'revolution' ? 'Glissez pour faire tourner' : activeLab === 'derivative' ? 'Glissez le point sur la courbe' : activeLab === 'riemann' ? 'Glissez pour ajouter des rectangles' : 'Ajustez l’ordre pour explorer' }}</div>
        </div>

        <aside class="controls-card playground-controls">
          <div class="control-heading"><div><span>À VOUS DE JOUER</span><h2>{{ currentPlay.controlTitle }}</h2></div><button class="reset-control" @click="resetPlayground" aria-label="Réinitialiser les réglages">↺</button></div>
          <template v-if="activeLab === 'derivative'">
            <div class="ctrl-group"><label>Une courbe</label><select v-model="d1.fn" @change="drawLab"><option value="parabola">Parabole douce</option><option value="cubic">Courbe cubique</option><option value="trig">Vague composée</option></select></div>
            <div class="ctrl-group"><label>Point de contact <strong>a = {{ d1.a.toFixed(2) }}</strong></label><input type="range" v-model.number="d1.a" min="-3" max="3" step="0.02" @input="drawLab" /></div>
            <div class="ctrl-group"><label>Écart <strong>h = {{ d1.h.toFixed(3) }}</strong></label><input type="range" v-model.number="d1.h" min="0.005" max="2.4" step="0.005" @input="drawLab" /></div>
            <button class="btn-primary full" @click="animateHToZero">Rapprocher les points <span>→</span></button>
          </template>
          <template v-else-if="activeLab === 'riemann'">
            <div class="ctrl-group"><label>Façon de mesurer</label><select v-model="d2.method" @change="drawLab"><option value="midpoint">Point milieu</option><option value="trapezoid">Trapèzes</option><option value="left">Depuis la gauche</option><option value="right">Depuis la droite</option></select></div>
            <div class="ctrl-group"><label>Découpage <strong>{{ d2.n }} morceaux</strong></label><input type="range" v-model.number="d2.n" min="2" max="90" step="1" @input="drawLab" /></div>
            <div class="ctrl-group"><label>Étendue <strong>b = {{ d2.b.toFixed(2) }}</strong></label><input type="range" v-model.number="d2.b" min="1.5" max="5.5" step="0.25" @input="drawLab" /></div>
          </template>
          <template v-else-if="activeLab === 'revolution'">
            <div class="ctrl-group"><label>Donner une forme</label><select v-model="d3.preset" @change="drawLab"><option value="paraboloid">Paraboloïde</option><option value="washer">Rondelle creuse</option><option value="vase">Vase de Venturi</option></select></div>
            <div class="ctrl-group"><label>Tourner de <strong>{{ d3.theta }}°</strong></label><input type="range" v-model.number="d3.theta" min="15" max="360" step="5" @input="drawLab" /></div>
            <div class="ctrl-group"><label>Déplacer la tranche <strong>x = {{ d3.sliceX.toFixed(2) }}</strong></label><input type="range" v-model.number="d3.sliceX" min="0.3" max="3.4" step="0.05" @input="drawLab" /></div>
          </template>
          <template v-else>
            <div class="ctrl-group"><label>Une fonction</label><select v-model="d4.fn" @change="drawLab"><option value="sin">sin(x)</option><option value="cos">cos(x)</option><option value="exp">eˣ</option><option value="geom">1 / (1 − x)</option><option value="ln">ln(1 + x)</option></select></div>
            <div class="ctrl-group"><label>Ordre d’approximation <strong>n = {{ d4.n }}</strong></label><input type="range" v-model.number="d4.n" min="0" max="15" step="1" @input="drawLab" /></div>
          </template>
          <div class="control-footnote">Les mathématiques répondent en direct à vos gestes.</div>
        </aside>
      </section>

      <section class="play-prompts"><div class="prompt-label">Essayez un petit défi</div><button v-for="prompt in playPrompts" :key="prompt.id" @click="runScenario(prompt.id)">{{ prompt.text }} <span>↗</span></button></section>
    </main>

    <footer class="footer">
      MAT145 — Calcul différentiel et intégral • École de technologie supérieure (ÉTS Montréal)
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, nextTick, computed } from 'vue';

const activeView = ref('studio');
const introVisible = ref(true);
const activeLab = ref('derivative');
const projectorMode = ref(false);

const introCanvas = ref(null);
const introPointer = reactive({ x: 0.72, y: 0.45 });
const labCanvas = ref(null);
const hudLines = ref([]);

let animFrame = null;
let heroTime = 0;

const d1 = reactive({ fn: 'parabola', a: 1.0, h: 1.2, dragging: false });
const d2 = reactive({ method: 'midpoint', n: 10, b: 4.0 });
const d3 = reactive({ preset: 'washer', theta: 260, sliceX: 2.0, yaw: 0.72, pitch: 0.35, dragging: false, lx: 0, ly: 0 });
const d4 = reactive({ fn: 'sin', n: 5 });

const playgroundModes = [
  { id: 'derivative', symbol: '⌁', label: 'Tangente' },
  { id: 'riemann', symbol: '▥', label: 'Aire' },
  { id: 'revolution', symbol: '◉', label: 'Rotation' },
  { id: 'taylor', symbol: '∿', label: 'Approximation' }
];
const playPrompts = [
  { id: 1, text: 'Trouver le point plat' },
  { id: 2, text: 'Découper sous la courbe' },
  { id: 3, text: 'Ouvrir une forme en deux' },
  { id: 4, text: 'Voir où la série décroche' }
];
const currentPlay = computed(() => ({
  derivative: { heading: 'Une tangente en mouvement', caption: 'Le point glisse. La sécante se redresse.', controlTitle: 'Trouver la pente' },
  riemann: { heading: 'Une aire qui se construit', caption: 'Ajoutez des morceaux. Regardez l’écart diminuer.', controlTitle: 'Construire une aire' },
  revolution: { heading: 'Une courbe prend du volume', caption: 'Faites tourner le profil et découpez une tranche.', controlTitle: 'Donner du volume' },
  taylor: { heading: 'Une fonction, plusieurs approches', caption: 'Les polynômes s’approchent… jusqu’à leur limite.', controlTitle: 'Approcher la courbe' }
}[activeLab.value]));

function enterPlayground() {
  introVisible.value = false;
  nextTick(() => drawLab());
}

function switchView(v) {
  activeView.value = v;
  if (v === 'studio') nextTick(() => drawLab());
}

function moveIntroArt(event) {
  const rect = event.currentTarget.getBoundingClientRect();
  introPointer.x = (event.clientX - rect.left) / rect.width;
  introPointer.y = (event.clientY - rect.top) / rect.height;
}

function selectLab(id) {
  activeLab.value = id;
  nextTick(() => drawLab());
}

function resetPlayground() {
  if (activeLab.value === 'derivative') Object.assign(d1, { fn: 'parabola', a: 1, h: 1.2 });
  if (activeLab.value === 'riemann') Object.assign(d2, { method: 'midpoint', n: 10, b: 4 });
  if (activeLab.value === 'revolution') Object.assign(d3, { preset: 'washer', theta: 260, sliceX: 2, yaw: 0.72, pitch: 0.35 });
  if (activeLab.value === 'taylor') Object.assign(d4, { fn: 'sin', n: 5 });
  nextTick(() => drawLab());
}

function runScenario(id) {
  if (id === 1) { d1.fn = 'cubic'; d1.a = 1.0; d1.h = 1.3; openLab('derivative'); }
  else if (id === 2) { d2.method = 'midpoint'; d2.n = 6; d2.b = 4.5; openLab('riemann'); }
  else if (id === 3) { d3.preset = 'washer'; d3.theta = 240; d3.sliceX = 2.0; openLab('revolution'); }
  else if (id === 4) { d4.fn = 'geom'; d4.n = 11; openLab('taylor'); }
}

function animateHero() {
  const art = introCanvas.value;
  if (introVisible.value && art) {
    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const w = art.clientWidth, h = art.clientHeight;
    if (art.width !== Math.round(w * dpr) || art.height !== Math.round(h * dpr)) { art.width = Math.round(w * dpr); art.height = Math.round(h * dpr); }
    const ctx = art.getContext('2d'); ctx.setTransform(dpr, 0, 0, dpr, 0, 0); ctx.clearRect(0, 0, w, h);
    const ox = w * (0.69 + (introPointer.x - 0.5) * 0.12), oy = h * (0.49 + (introPointer.y - 0.5) * 0.1);
    const scale = Math.min(w, h) * 0.29;
    const colors = ['#5b8cff', '#a28bff', '#e1a7dc', '#86c9e8', '#9bd8cb'];
    for (let layer = 0; layer < 5; layer++) {
      ctx.beginPath();
      for (let i = 0; i <= 560; i++) {
        const t = i / 560 * Math.PI * 2;
        const wave = 1 + 0.18 * Math.sin(3 * t + heroTime * 0.7 + layer * 0.24) + 0.08 * Math.cos(5 * t - heroTime * 0.38);
        const r = scale * wave * (0.48 + layer * 0.105);
        const x = ox + Math.cos(t + layer * 0.11 + heroTime * 0.05) * r;
        const y = oy + Math.sin(t) * r * 0.76 + Math.sin(2 * t + heroTime * 0.28) * scale * 0.035;
        if (i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
      }
      ctx.closePath(); ctx.strokeStyle = colors[layer]; ctx.globalAlpha = 0.24 + layer * 0.085; ctx.lineWidth = layer === 2 ? 1.8 : 1; ctx.stroke();
    }
    ctx.globalAlpha = 1;
    const glow = ctx.createRadialGradient(ox, oy, 0, ox, oy, scale * 0.9); glow.addColorStop(0, 'rgba(173,190,255,0.17)'); glow.addColorStop(1, 'rgba(173,190,255,0)'); ctx.fillStyle = glow; ctx.fillRect(ox-scale, oy-scale, scale*2, scale*2);
    for (let i=0;i<8;i++){ const t=heroTime*0.22+i*Math.PI/4; const r=scale*(0.62+0.055*Math.sin(heroTime+i)); ctx.beginPath(); ctx.arc(ox+Math.cos(t)*r,oy+Math.sin(t)*r*0.76,2.2,0,Math.PI*2); ctx.fillStyle=colors[i%5];ctx.globalAlpha=.8;ctx.fill(); } ctx.globalAlpha=1;
  }
  heroTime += 0.022;
  animFrame = requestAnimationFrame(animateHero);
}

function fact(n) { let r = 1; for (let i = 2; i <= n; i++) r *= i; return r; }

function drawLab() {
  const c = labCanvas.value;
  if (!c) return;
  const dpr = window.devicePixelRatio || 1;
  const w = c.clientWidth || 700, h = c.clientHeight || 540;
  c.width = w * dpr; c.height = h * dpr;
  const ctx = c.getContext('2d');
  ctx.resetTransform();
  ctx.scale(dpr, dpr);
  ctx.fillStyle = '#f1f2f8';
  ctx.fillRect(0, 0, w, h);

  if (activeLab.value === 'derivative') {
    const fns = {
      parabola: { f: x => x * x - 2 * x, df: x => 2 * x - 2 },
      cubic: { f: x => 0.25 * Math.pow(x, 3) - 0.75 * x + 0.5, df: x => 0.75 * x * x - 0.75 },
      trig: { f: x => 1.8 * Math.sin(x) + 0.4 * Math.cos(2 * x), df: x => 1.8 * Math.cos(x) - 0.8 * Math.sin(2 * x) }
    };
    const fn = fns[d1.fn];
    const ox = w / 2, oy = h / 2 + 20, sx = w / 9.5, sy = h / 7;
    const toX = x => ox + x * sx, toY = y => oy - y * sy, fromX = px => (px - ox) / sx;

    ctx.strokeStyle = '#e3e4eb';
    for (let x = -5; x <= 5; x++) { ctx.beginPath(); ctx.moveTo(toX(x), 0); ctx.lineTo(toX(x), h); ctx.stroke(); }
    for (let y = -4; y <= 4; y++) { ctx.beginPath(); ctx.moveTo(0, toY(y)); ctx.lineTo(w, toY(y)); ctx.stroke(); }
    ctx.strokeStyle = '#b8bac6'; ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(0, oy); ctx.lineTo(w, oy); ctx.moveTo(ox, 0); ctx.lineTo(ox, h); ctx.stroke();

    const fa = fn.f(d1.a), fah = fn.f(d1.a + d1.h);
    const mSec = (fah - fa) / d1.h, mTan = fn.df(d1.a);

    ctx.strokeStyle = '#7483cb'; ctx.lineWidth = 3;
    ctx.beginPath();
    for (let px = 0; px <= w; px += 2) {
      const py = toY(fn.f(fromX(px)));
      if (px === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
    }
    ctx.stroke();

    const xL = fromX(0), xR = fromX(w);
    ctx.strokeStyle = '#c18d41'; ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(0, toY(fa + mSec * (xL - d1.a))); ctx.lineTo(w, toY(fa + mSec * (xR - d1.a))); ctx.stroke();

    ctx.strokeStyle = '#5a9b79'; ctx.setLineDash([6, 4]);
    ctx.beginPath(); ctx.moveTo(0, toY(fa + mTan * (xL - d1.a))); ctx.lineTo(w, toY(fa + mTan * (xR - d1.a))); ctx.stroke();
    ctx.setLineDash([]);

    ctx.fillStyle = '#c18d41'; ctx.beginPath(); ctx.arc(toX(d1.a + d1.h), toY(fah), 5.5, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#d57282'; ctx.beginPath(); ctx.arc(toX(d1.a), toY(fa), 7, 0, Math.PI * 2); ctx.fill();

    hudLines.value = [
      { label: 'Abscisse a :', value: d1.a.toFixed(3), color: '#f87171' },
      { label: 'Pas h (Δx) :', value: d1.h.toFixed(3), color: '#c18d41' },
      { label: 'Pente sécante :', value: mSec.toFixed(4), color: '#c18d41' },
      { label: "Tangente f'(a) :", value: mTan.toFixed(4), color: '#7483cb' },
      { label: 'Erreur :', value: Math.abs(mSec - mTan).toFixed(5), color: '#5a9b79' }
    ];
  } else if (activeLab.value === 'riemann') {
    const fn = x => Math.sin(x) + 1.4;
    const F = x => -Math.cos(x) + 1.4 * x;
    const padL = 50, padR = 30, padT = 35, padB = 45;
    const pw = w - padL - padR, ph = h - padT - padB;
    const xMin = -0.3, xMax = 6.0, yMin = -0.2, yMax = 3.0;
    const toX = x => padL + ((x - xMin) / (xMax - xMin)) * pw;
    const toY = y => padT + (1 - (y - yMin) / (yMax - yMin)) * ph;

    const dx = d2.b / d2.n, y0 = toY(0);
    let sum = 0;
    for (let i = 0; i < d2.n; i++) {
      const x0 = i * dx, x1 = x0 + dx;
      const sx0 = toX(x0), sx1 = toX(x1);
      if (d2.method === 'trapezoid') {
        const yA = fn(x0), yB = fn(x1);
        sum += 0.5 * (yA + yB) * dx;
        ctx.fillStyle = 'rgba(251, 191, 36, 0.25)';
        ctx.beginPath(); ctx.moveTo(sx0, y0); ctx.lineTo(sx0, toY(yA)); ctx.lineTo(sx1, toY(yB)); ctx.lineTo(sx1, y0); ctx.closePath(); ctx.fill();
        ctx.strokeStyle = '#c18d41'; ctx.stroke();
      } else {
        const xs = d2.method === 'left' ? x0 : (d2.method === 'right' ? x1 : 0.5 * (x0 + x1));
        const ys = fn(xs); sum += ys * dx;
        const sy = toY(ys);
        ctx.fillStyle = 'rgba(116, 131, 203, 0.22)';
        ctx.fillRect(sx0, Math.min(sy, y0), sx1 - sx0, Math.abs(y0 - sy));
        ctx.strokeStyle = '#7483cb'; ctx.strokeRect(sx0, Math.min(sy, y0), sx1 - sx0, Math.abs(y0 - sy));
      }
    }
    ctx.strokeStyle = '#3f4048'; ctx.lineWidth = 3;
    ctx.beginPath();
    for (let px = padL; px <= w - padR; px += 2) {
      const x = xMin + ((px - padL) / pw) * (xMax - xMin);
      const py = toY(fn(x));
      if (px === padL) ctx.moveTo(px, py); else ctx.lineTo(px, py);
    }
    ctx.stroke();

    const exact = F(d2.b) - F(0);
    hudLines.value = [
      { label: 'Subdivisions n :', value: d2.n, color: '#c18d41' },
      { label: 'Somme S_n :', value: sum.toFixed(4), color: '#c18d41' },
      { label: 'Intégrale exacte :', value: exact.toFixed(4), color: '#7483cb' },
      { label: 'Erreur |I - S_n| :', value: Math.abs(exact - sum).toFixed(5), color: '#5a9b79' }
    ];
  } else if (activeLab.value === 'revolution') {
    const presets = {
      paraboloid: { R: x => Math.sqrt(Math.max(0, x)), r: () => 0, b: 3.8, vol: 22.68 },
      washer: { R: x => Math.sqrt(Math.max(0, x)) + 0.6, r: x => 0.25 * x, b: 3.6, vol: 41.85 },
      vase: { R: x => 1.3 + 0.55 * Math.cos(1.5 * x), r: () => 0.35, b: 3.8, vol: 19.8 }
    };
    const p = presets[d3.preset];
    const cx = w * 0.46, cy = h * 0.54, scale = Math.min(w, h) / 5.4;
    const proj = (x, y, z) => {
      const cyw = Math.cos(d3.yaw), syw = Math.sin(d3.yaw);
      const x1 = x * cyw + z * syw, z1 = -x * syw + z * cyw;
      const cp = Math.cos(d3.pitch), sp = Math.sin(d3.pitch);
      const y2 = y * cp - z1 * sp, z2 = y * sp + z1 * cp;
      const fov = 9.5 / (9.5 + z2);
      return { px: cx + x1 * scale * fov, py: cy - y2 * scale * fov, depth: z2 };
    };

    const maxRad = (d3.theta * Math.PI) / 180;
    const nx = 20, nt = Math.max(6, Math.floor((d3.theta / 360) * 28));
    const faces = [];
    for (let i = 0; i < nx; i++) {
      const x0 = (i / nx) * p.b, x1 = ((i + 1) / nx) * p.b;
      const R0 = p.R(x0), R1 = p.R(x1), r0 = p.r(x0), r1 = p.r(x1);
      for (let j = 0; j < nt; j++) {
        const t0 = (j / nt) * maxRad, t1 = ((j + 1) / nt) * maxRad;
        const pA = proj(x0, R0 * Math.cos(t0), R0 * Math.sin(t0));
        const pB = proj(x1, R1 * Math.cos(t0), R1 * Math.sin(t0));
        const pC = proj(x1, R1 * Math.cos(t1), R1 * Math.sin(t1));
        const pD = proj(x0, R0 * Math.cos(t1), R0 * Math.sin(t1));
        faces.push({ pts: [pA, pB, pC, pD], depth: (pA.depth + pC.depth) * 0.5, shade: 0.5 + 0.45 * Math.cos(t0), inner: false });
        if (r0 > 0.01 || r1 > 0.01) {
          const qA = proj(x0, r0 * Math.cos(t0), r0 * Math.sin(t0));
          const qB = proj(x1, r1 * Math.cos(t0), r1 * Math.sin(t0));
          const qC = proj(x1, r1 * Math.cos(t1), r1 * Math.sin(t1));
          const qD = proj(x0, r0 * Math.cos(t1), r0 * Math.sin(t1));
          faces.push({ pts: [qA, qB, qC, qD], depth: (qA.depth + qC.depth) * 0.5, shade: 0.4, inner: true });
        }
      }
    }
    faces.sort((a, b) => b.depth - a.depth);
    faces.forEach(f => {
      ctx.beginPath();
      ctx.moveTo(f.pts[0].px, f.pts[0].py);
      f.pts.slice(1).forEach(pt => ctx.lineTo(pt.px, pt.py));
      ctx.closePath();
      const col = Math.floor(140 + 100 * f.shade);
      ctx.fillStyle = f.inner ? 'rgba(192, 132, 252, 0.35)' : `rgba(35, ${Math.floor(col * 0.72)}, ${col}, 0.62)`;
      ctx.fill();
      ctx.strokeStyle = 'rgba(56, 189, 248, 0.2)'; ctx.lineWidth = 0.7; ctx.stroke();
    });

    const Rk = p.R(d3.sliceX), rk = p.r(d3.sliceX);
    ctx.beginPath();
    for (let j = 0; j <= 32; j++) {
      const t = (j / 32) * maxRad;
      const pt = proj(d3.sliceX, Rk * Math.cos(t), Rk * Math.sin(t));
      if (j === 0) ctx.moveTo(pt.px, pt.py); else ctx.lineTo(pt.px, pt.py);
    }
    for (let j = 32; j >= 0; j--) {
      const t = (j / 32) * maxRad;
      const pt = proj(d3.sliceX, rk * Math.cos(t), rk * Math.sin(t));
      ctx.lineTo(pt.px, pt.py);
    }
    ctx.closePath();
    ctx.fillStyle = 'rgba(239, 62, 66, 0.75)'; ctx.fill();
    ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.stroke();

    hudLines.value = [
      { label: 'Angle θ :', value: d3.theta + '°', color: '#c18d41' },
      { label: 'Rayon R(x_k) :', value: Rk.toFixed(3), color: '#7483cb' },
      { label: 'Rayon r(x_k) :', value: rk.toFixed(3), color: '#9183c7' },
      { label: 'Volume V(θ) :', value: ((d3.theta / 360) * p.vol).toFixed(2) + ' u³', color: '#5a9b79' }
    ];
  } else {
    const ox = w / 2, oy = h / 2, sx = w / 11, sy = h / 6.5;
    const toX = x => ox + x * sx, toY = y => oy - y * sy, fromX = px => (px - ox) / sx;
    ctx.strokeStyle = '#b8bac6'; ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(0, oy); ctx.lineTo(w, oy); ctx.moveTo(ox, 0); ctx.lineTo(ox, h); ctx.stroke();

    const evalF = x => d4.fn === 'sin' ? Math.sin(x) : d4.fn === 'cos' ? Math.cos(x) : d4.fn === 'exp' ? Math.exp(x) : d4.fn === 'geom' ? 1 / (1 - x) : Math.log(1 + x);
    const evalP = (x, n) => {
      let s = 0;
      for (let k = 0; k <= n; k++) {
        if (d4.fn === 'sin') s += [0, 1, 0, -1][k % 4] * Math.pow(x, k) / fact(k);
        else if (d4.fn === 'cos') s += [1, 0, -1, 0][k % 4] * Math.pow(x, k) / fact(k);
        else if (d4.fn === 'exp') s += Math.pow(x, k) / fact(k);
        else if (d4.fn === 'geom') s += Math.pow(x, k);
        else if (k >= 1) s += (k % 2 === 1 ? 1 : -1) * Math.pow(x, k) / k;
      }
      return s;
    };

    ctx.strokeStyle = '#7483cb'; ctx.lineWidth = 3;
    ctx.beginPath();
    let st = false;
    for (let px = 0; px <= w; px += 2) {
      const x = fromX(px), y = evalF(x);
      if (isFinite(y) && Math.abs(y) < 10) {
        if (!st) { ctx.moveTo(px, toY(y)); st = true; } else ctx.lineTo(px, toY(y));
      } else st = false;
    }
    ctx.stroke();

    ctx.strokeStyle = '#c18d41'; ctx.lineWidth = 2.5;
    ctx.beginPath(); st = false;
    for (let px = 0; px <= w; px += 2) {
      const x = fromX(px), y = evalP(x, d4.n);
      if (isFinite(y) && Math.abs(y) < 12) {
        if (!st) { ctx.moveTo(px, toY(y)); st = true; } else ctx.lineTo(px, toY(y));
      } else st = false;
    }
    ctx.stroke();

    hudLines.value = [
      { label: 'Ordre Taylor n :', value: d4.n, color: '#c18d41' },
      { label: 'Rayon R :', value: (d4.fn === 'geom' || d4.fn === 'ln') ? 'R = 1' : 'R = ∞', color: '#5a9b79' }
    ];
  }
}

function animateHToZero() {
  d1.h = 2.2;
  function step() {
    d1.h *= 0.93;
    if (d1.h <= 0.008) { d1.h = 0.005; drawLab(); return; }
    drawLab();
    requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}

function onCanvasDown(e) {
  e.currentTarget.setPointerCapture(e.pointerId);
  if (activeLab.value === 'derivative') d1.dragging = true;
  if (activeLab.value === 'riemann') d2.dragging = true;
  if (activeLab.value === 'revolution') { d3.dragging = true; d3.lx = e.clientX; d3.ly = e.clientY; }
  onCanvasMove(e);
}
function onCanvasMove(e) {
  const rect = e.currentTarget.getBoundingClientRect();
  const position = Math.max(0, Math.min(1, (e.clientX - rect.left) / rect.width));
  if (activeLab.value === 'derivative' && d1.dragging) {
    d1.a = Math.max(-3, Math.min(3, (position - 0.5) * 9.5));
    drawLab();
  } else if (activeLab.value === 'riemann' && d2.dragging) {
    d2.n = Math.round(2 + position * 88);
    drawLab();
  } else if (activeLab.value === 'revolution' && d3.dragging) {
    d3.yaw += (e.clientX - d3.lx) * 0.008;
    d3.pitch = Math.max(-1.2, Math.min(1.2, d3.pitch - (e.clientY - d3.ly) * 0.008));
    d3.lx = e.clientX; d3.ly = e.clientY;
    drawLab();
  }
}
function onCanvasUp() { d1.dragging = false; d2.dragging = false; d3.dragging = false; }

function exportPNG() {
  if (!labCanvas.value) return;
  const a = document.createElement('a');
  a.download = `ets-mat145-${activeLab.value}.png`;
  a.href = labCanvas.value.toDataURL('image/png');
  a.click();
}

onMounted(() => {
  animFrame = requestAnimationFrame(animateHero);
});
onUnmounted(() => {
  if (animFrame) cancelAnimationFrame(animFrame);
});
</script>

<style scoped>
:global(*) { box-sizing: border-box; }
:global(body) { margin: 0; background: #f5f5f7; color: #1d1d1f; font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Helvetica Neue", sans-serif; -webkit-font-smoothing: antialiased; }
:global(button), :global(select), :global(input) { font: inherit; }
.app-shell { --ink:#1d1d1f; --muted:#6e6e73; --line:rgba(29,29,31,.1); --blue:#1677ff; min-height:100vh; display:flex; flex-direction:column; background:#f5f5f7; color:var(--ink); }
.app-shell.projector { font-size:112%; }
.intro-page { min-height:100svh; position:fixed;inset:0;z-index:100;overflow:hidden;isolation:isolate; background:radial-gradient(ellipse at 74% 46%,#f0f1fb 0%,#f6f6f8 42%,#f5f5f7 76%); }
.intro-art { position:absolute; inset:0; width:100%; height:100%; z-index:0; touch-action:pan-y; }
.intro-topline { position:absolute;z-index:1; left:clamp(24px,5vw,76px); right:clamp(24px,5vw,76px); top:34px; display:flex; align-items:center; gap:12px; color:#56565c; font-size:13px; letter-spacing:.02em; }
.intro-mark { width:30px;height:30px;border-radius:50%;display:grid;place-items:center;background:#1d1d1f;color:white;font-family:Georgia,serif;font-size:20px; }
.intro-topline i { color:#aaa; font-style:normal; padding:0 3px; }
.intro-live { margin-left:auto; display:flex;align-items:center;gap:8px;color:#77777d;font-size:12px; }
.intro-live b { width:6px;height:6px;border-radius:50%;background:#65b98c;box-shadow:0 0 0 4px rgba(101,185,140,.12); }
.intro-content { position:absolute;z-index:1;left:clamp(28px,11.2vw,172px);top:50%;transform:translateY(-48%);width:min(530px,50vw); }
.intro-kicker { color:#7a7a82;font-size:11px;font-weight:650;letter-spacing:.17em;margin-bottom:23px; }
.intro-content h1 { font-size:clamp(54px,7.4vw,100px);line-height:.98;letter-spacing:-.075em;font-weight:650;margin:0;color:#222226; }
.intro-content h1 em { font-style:normal;color:#8e8fd4; }
.intro-content p { font-size:18px;line-height:1.55;color:#6e6e73;max-width:390px;margin:25px 0 28px; }
.intro-enter { border:0;border-radius:999px;background:#1d1d1f;color:white;padding:15px 22px 15px 24px;font-size:14px;font-weight:600;cursor:pointer;box-shadow:0 9px 25px rgba(29,29,31,.13);transition:transform .2s,background .2s; }
.intro-enter span { padding-left:17px;font-size:17px; }
.intro-enter:hover { transform:translateY(-2px);background:#353539; }
.intro-hint { display:flex;align-items:center;gap:10px;color:#929299;font-size:11px;margin-top:24px;letter-spacing:.015em; }
.hint-line { width:24px;height:1px;background:#b7b7be; }
.intro-equation { position:absolute;z-index:1;color:rgba(75,79,117,.48);font-family:Georgia,serif;font-size:20px;letter-spacing:.02em; }
.equation-one { top:24%;right:17%;transform:rotate(-8deg); }
.equation-two { bottom:26%;right:8%;transform:rotate(7deg);font-size:17px; }
.intro-index { position:absolute;z-index:1;right:clamp(28px,5vw,76px);top:50%;transform:translateY(-50%);display:flex;align-items:center;gap:12px;color:#8b8b91;font-size:10px;letter-spacing:.12em;writing-mode:vertical-rl; }
.intro-index span:first-child { color:#414148;font-size:12px; }
.index-rule { height:62px;width:1px;background:linear-gradient(#aaa,#ddd); }
.intro-bottom { position:absolute;z-index:1;left:clamp(24px,5vw,76px);right:clamp(24px,5vw,76px);bottom:27px;display:flex;justify-content:space-between;color:#929299;font-size:11px;letter-spacing:.02em; }
.intro-bottom b { font-weight:400;padding-left:6px; }
.navbar { position:sticky;top:0;z-index:50;background:rgba(250,250,252,.78);backdrop-filter:blur(20px);border-bottom:1px solid rgba(29,29,31,.08);padding:12px 26px; }
.nav-inner { max-width:1360px;margin:0 auto;display:flex;flex-wrap:wrap;align-items:center;justify-content:space-between;gap:16px; }
.brand { display:flex;align-items:center;gap:11px;cursor:pointer; }
.ets-logo { background:#1d1d1f;color:#fff;font-size:13px;letter-spacing:-.04em;padding:8px 10px;border-radius:10px;font-weight:750; }
.brand-title { font-weight:650;font-size:15px;display:flex;align-items:center;gap:8px;letter-spacing:-.02em; }
.badge { font-size:10px;color:#77777d;background:#f0f0f2;padding:4px 7px;border-radius:99px;font-weight:500; }
.brand-sub { font-size:11px;color:#8a8a90;margin-top:2px; }
.nav-tabs { display:flex;gap:3px;background:#ededf0;padding:4px;border-radius:99px; }
.nav-btn { background:transparent;border:0;color:#66666c;padding:9px 14px;border-radius:99px;font-size:12px;font-weight:550;cursor:pointer; }
.nav-btn.active { background:white;color:#1d1d1f;box-shadow:0 1px 4px rgba(0,0,0,.09); }
.btn-outline { background:rgba(255,255,255,.8);border:1px solid #d9d9de;color:#44444b;padding:9px 14px;border-radius:99px;font-size:12px;font-weight:550;cursor:pointer; }
.btn-outline.active { border-color:#9bbdf5;color:#1769d1;background:#f3f8ff; }
.main-content { flex:1; }
.hero-section { position:relative;overflow:hidden;padding:74px 30px 60px;border-bottom:1px solid var(--line);background:radial-gradient(ellipse at 70% 38%,#e8eafa 0%,#f5f5f7 40%,#f5f5f7 78%); }
.hero-bg-canvas { position:absolute;inset:0;width:100%;height:100%;pointer-events:none;opacity:.5; }
.hero-grid { max-width:1280px;margin:0 auto;display:grid;grid-template-columns:1.05fr .95fr;gap:70px;align-items:center;position:relative;z-index:2; }
.hero-pill { display:inline-flex;align-items:center;gap:8px;color:#777780;padding:7px 12px;border-radius:99px;font-size:10px;font-weight:600;letter-spacing:.09em;margin-bottom:21px; }
.dot { width:6px;height:6px;background:#8791d5;border-radius:50%; }
.hero-h1 { font-size:clamp(38px,5vw,66px);font-weight:650;line-height:1.02;letter-spacing:-.065em;margin:0 0 20px; }
.grad-text { color:#8586c9;background:none;-webkit-text-fill-color:currentColor; }
.hero-p { color:#6e6e73;font-size:16px;line-height:1.65;margin:0 0 25px;max-width:620px; }
.hero-actions { display:flex;flex-wrap:wrap;gap:10px;margin-bottom:34px; }
.btn-primary { background:#1d1d1f;color:white;border:0;padding:13px 19px;border-radius:99px;font-weight:600;font-size:13px;cursor:pointer;box-shadow:0 5px 16px rgba(0,0,0,.1); }
.btn-primary:hover { background:#3a3a3f; }
.btn-primary.full { width:100%;margin-top:8px; }
.btn-secondary { background:rgba(255,255,255,.7);color:#37373d;border:1px solid #dddde2;padding:13px 18px;border-radius:99px;font-weight:550;font-size:13px;cursor:pointer; }
.metrics-row { display:grid;grid-template-columns:repeat(4,1fr);gap:12px;padding-top:20px;border-top:1px solid var(--line); }
.metrics-row strong { display:block;font-size:19px;font-weight:620;letter-spacing:-.04em; }
.metrics-row span { font-size:10px;color:#828289; }
.hero-card { background:rgba(255,255,255,.57);border:1px solid rgba(255,255,255,.8);border-radius:24px;padding:16px;box-shadow:0 24px 70px rgba(66,68,104,.12);backdrop-filter:blur(18px); }
.card-header,.card-footer { display:flex;justify-content:space-between;font-size:10px;color:#777780; }
.card-header { margin-bottom:10px;padding:3px 3px 10px;border-bottom:1px solid rgba(29,29,31,.08); }
.card-footer { margin-top:10px;color:#8c8c92; }
.status-ok { color:#5b9c74; }
.mini-canvas { width:100%;height:260px;background:linear-gradient(145deg,#f4f5fb,#ebeefa);border-radius:15px;display:block; }
.bento-section { max-width:1280px;margin:0 auto;padding:54px 30px;width:100%; }
.sec-head { text-align:center;margin-bottom:30px; }
.eyebrow { font-size:10px;letter-spacing:.14em;color:#8586c9;display:block;margin-bottom:9px;font-weight:650; }
.sec-head h2 { font-size:30px;font-weight:650;letter-spacing:-.05em;margin:0 0 8px; }
.sec-head p { color:#77777e;font-size:14px; }
.bento-grid { display:grid;grid-template-columns:repeat(2,1fr);gap:14px; }
.bento-card { background:rgba(255,255,255,.7);border:1px solid rgba(29,29,31,.07);border-radius:20px;padding:23px;cursor:pointer;display:flex;flex-direction:column;justify-content:space-between;min-height:225px;transition:transform .2s,box-shadow .2s,border-color .2s; }
.bento-card:hover { transform:translateY(-3px);border-color:#d3d4ee;box-shadow:0 14px 36px rgba(43,45,76,.08); }
.bento-meta { display:flex;justify-content:space-between;margin-bottom:13px; }
.mod-badge { font-size:9px;font-weight:650;letter-spacing:.08em;color:#7476b9;background:#f0f0fa;padding:6px 9px;border-radius:99px; }
.mod-weeks { font-size:11px;color:#929299; }
.bento-card h3 { font-size:18px;letter-spacing:-.035em;margin:0 0 8px;font-weight:620; }
.bento-card p { color:#75757c;font-size:13px;line-height:1.55;margin:0 0 14px; }
.formula-box { font-family:ui-monospace,SFMono-Regular,Menlo,monospace;font-size:11px;background:#f6f6fa;border:1px solid #ededf2;padding:10px 12px;border-radius:10px;color:#7173ad;margin-bottom:14px; }
.bento-link { display:flex;justify-content:space-between;padding-top:11px;border-top:1px solid rgba(29,29,31,.08);color:#6264aa;font-weight:600;font-size:12px; }
.studio-container { max-width:1380px;margin:0 auto;padding:27px 30px;width:100%; }
.studio-bar { display:flex;flex-wrap:wrap;justify-content:space-between;gap:12px;margin-bottom:18px;padding-bottom:15px;border-bottom:1px solid var(--line); }
.lab-buttons { display:flex;flex-wrap:wrap;gap:7px; }
.lab-tab { background:rgba(255,255,255,.7);border:1px solid #e4e4e8;color:#66666c;padding:9px 14px;border-radius:99px;font-size:12px;font-weight:550;cursor:pointer; }
.lab-tab.active { border-color:#bfc0e6;color:#6769b2;background:#f3f3fc; }
.studio-grid { display:grid;grid-template-columns:1fr 320px;gap:16px; }
.stage-card,.controls-card { background:rgba(255,255,255,.7);border:1px solid rgba(29,29,31,.08);border-radius:20px;overflow:hidden;box-shadow:0 8px 28px rgba(34,35,55,.04); }
.stage-canvas { width:100%;height:540px;display:block;background:#f1f2f8;cursor:crosshair; }
.hud-box { position:absolute;top:14px;left:14px;background:rgba(255,255,255,.88);border:1px solid rgba(29,29,31,.1);border-radius:12px;padding:12px 14px;font-size:11px;min-width:210px;pointer-events:none;box-shadow:0 4px 20px rgba(0,0,0,.05); }
.hud-line { display:flex;justify-content:space-between;gap:16px;margin-bottom:4px;color:#707078; }
.controls-card { padding:19px; }
.controls-card h3 { font-size:14px;font-weight:620;padding-bottom:12px;margin:0 0 16px;border-bottom:1px solid var(--line);letter-spacing:-.02em; }
.ctrl-group { margin-bottom:16px; }
.ctrl-group label { display:block;font-size:12px;color:#66666d;margin-bottom:7px; }
.ctrl-group select,.ctrl-group input[type=range] { width:100%; }
.ctrl-group select { background:#fafafd;color:#303036;border:1px solid #dedee4;padding:9px;border-radius:10px;font-size:12px; }
.ctrl-group input[type=range] { accent-color:#8586c9; }
.hint { font-size:11px;color:#83838a;line-height:1.5; }
.footer { margin-top:auto;border-top:1px solid var(--line);padding:17px;text-align:center;font-size:11px;color:#929299; }
@media(max-width:960px) { .hero-grid{grid-template-columns:1fr;gap:32px}.intro-content{width:65vw}.studio-grid{grid-template-columns:1fr}.stage-canvas{height:460px} }
@media(max-width:700px) { .navbar{padding:10px 15px}.nav-inner{gap:10px}.nav-tabs{order:3;width:100%;justify-content:center}.nav-btn{padding:8px 10px;font-size:11px}.brand-sub,.badge{display:none}.hero-section{padding:55px 19px 42px}.hero-grid{gap:28px}.metrics-row{grid-template-columns:repeat(2,1fr);row-gap:16px}.mini-canvas{height:210px}.bento-section{padding:40px 18px}.bento-grid{grid-template-columns:1fr}.studio-container{padding:20px 15px}.stage-canvas{height:380px}.intro-content{left:28px;top:44%;width:80vw}.intro-content h1{font-size:clamp(52px,13vw,78px)}.intro-content p{font-size:16px;max-width:310px}.intro-topline{top:22px;left:22px;right:22px}.intro-live{font-size:10px}.intro-index{right:17px;font-size:8px}.equation-one{top:20%;right:14%;font-size:16px}.equation-two{bottom:23%;right:9%;font-size:14px}.intro-bottom{left:22px;right:22px;bottom:20px;font-size:9px}.intro-bottom span:first-child{max-width:55%} }
@media(prefers-reduced-motion:reduce) { *,*::before,*::after{scroll-behavior:auto!important;transition:none!important} }

/* Canvas-first playground */
.header-note { display:flex;align-items:center;gap:9px;color:#77777f;font-size:12px;letter-spacing:.01em; }
.header-note-dot { width:7px;height:7px;border-radius:50%;background:#8d90d3;box-shadow:0 0 0 4px rgba(141,144,211,.12); }
.app-shell.intro-active .navbar,.app-shell.intro-active .playground-shell,.app-shell.intro-active .footer { display:none; }
.playground-shell { flex:1;width:min(1450px,100%);margin:0 auto;padding:32px clamp(16px,3.3vw,48px) 24px; }
.playground-heading { display:flex;justify-content:space-between;align-items:flex-end;gap:24px;margin:2px 2px 18px; }
.play-eyebrow { display:flex;align-items:center;gap:8px;color:#898991;font-size:10px;font-weight:650;letter-spacing:.14em;margin-bottom:8px; }
.play-eyebrow span { width:6px;height:6px;background:#8c8ed0;border-radius:50%; }
.playground-heading h1 { margin:0;font-size:clamp(28px,3.4vw,43px);font-weight:650;letter-spacing:-.06em;line-height:1.06; }
.playground-heading p { margin:8px 0 0;color:#77777f;font-size:13px; }
.play-modes { display:flex;align-items:center;gap:5px;background:#eaeaf0;border:1px solid rgba(30,30,38,.035);padding:5px;border-radius:999px;flex-shrink:0; }
.play-mode { display:inline-flex;align-items:center;gap:7px;border:0;border-radius:999px;padding:9px 12px;background:transparent;color:#777780;font-size:11px;font-weight:550;cursor:pointer;transition:background .18s,color .18s,box-shadow .18s; }
.play-mode:hover { color:#414149; }
.play-mode.active { background:#fff;color:#29292f;box-shadow:0 2px 6px rgba(30,30,40,.08); }
.mode-symbol { color:#8587c8;font-family:Georgia,serif;font-size:15px;line-height:1; }
.play-stage-wrap { position:relative; }
.playground-stage { border-radius:22px;min-height:450px; }
.playground-stage .stage-canvas { height:min(63vh,700px);min-height:450px;background:radial-gradient(ellipse at 48% 48%,#fafaff,#eef0f8 70%,#edeef5);touch-action:pan-y;cursor:grab; }
.playground-stage .stage-canvas:active { cursor:grabbing; }
.scene-title { position:absolute;left:22px;top:20px;display:flex;align-items:flex-start;gap:10px;pointer-events:none; }
.scene-live { margin-top:7px;width:7px;height:7px;border-radius:50%;background:#73b990;box-shadow:0 0 0 4px rgba(115,185,144,.12); }
.scene-title strong,.scene-title small { display:block; }
.scene-title strong { font-size:13px;font-weight:620;letter-spacing:-.02em; }
.scene-title small { margin-top:3px;font-size:11px;color:#85858d; }
.stage-export { position:absolute;right:18px;top:18px;border:1px solid rgba(40,40,50,.08);border-radius:999px;background:rgba(255,255,255,.8);backdrop-filter:blur(12px);color:#5e5e67;padding:8px 11px;font-size:12px;cursor:pointer; }
.stage-export span { font-size:10px;padding-left:3px; }
.playground-stage .hud-box { top:auto;bottom:18px;left:18px;min-width:220px;background:rgba(255,255,255,.76);backdrop-filter:blur(16px);border-color:rgba(40,40,50,.09); }
.drag-tip { position:absolute;bottom:22px;right:22px;color:#85858d;font-size:10px;pointer-events:none; }
.playground-controls { position:absolute;right:18px;top:72px;width:276px;padding:17px;background:rgba(255,255,255,.78);backdrop-filter:blur(20px);border-color:rgba(255,255,255,.86);box-shadow:0 10px 34px rgba(50,52,85,.09); }
.control-heading { display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:16px; }
.control-heading span { color:#92929a;font-size:9px;font-weight:650;letter-spacing:.13em; }
.control-heading h2 { font-size:16px;margin:5px 0 0;font-weight:620;letter-spacing:-.035em; }
.reset-control { width:28px;height:28px;border:1px solid #e5e5eb;background:#fafafd;color:#767680;border-radius:50%;font-size:17px;line-height:1;cursor:pointer; }
.playground-controls .ctrl-group { margin-bottom:14px; }
.playground-controls .ctrl-group label { display:flex;align-items:center;justify-content:space-between;gap:8px;font-size:11px;color:#6f6f77; }
.playground-controls .ctrl-group label strong { color:#383840;font-size:11px;font-weight:600; }
.playground-controls .ctrl-group select { background:rgba(250,250,253,.8); }
.playground-controls .btn-primary { width:100%;margin-top:2px;padding:11px 14px;font-size:11px; }
.playground-controls .btn-primary span { margin-left:8px; }
.control-footnote { border-top:1px solid rgba(29,29,31,.08);padding-top:11px;margin-top:13px;color:#92929a;font-size:10px;line-height:1.45; }
.play-prompts { display:flex;align-items:center;gap:7px;flex-wrap:wrap;padding:16px 2px 0; }
.prompt-label { color:#92929a;font-size:10px;letter-spacing:.06em;margin-right:5px; }
.play-prompts button { display:inline-flex;align-items:center;gap:9px;border:1px solid rgba(29,29,31,.07);background:rgba(255,255,255,.62);border-radius:999px;padding:8px 11px;color:#696970;font-size:10px;cursor:pointer;transition:background .15s,border-color .15s; }
.play-prompts button:hover { background:white;border-color:#cccee8; }
.play-prompts button span { color:#8b8dc9; }
.playground-shell ~ .footer { display:none; }
@media(max-width:950px) { .playground-heading{align-items:flex-start;flex-direction:column;gap:15px}.playground-controls{width:255px;right:12px}.playground-stage .stage-canvas{height:62vh;min-height:440px} }
@media(max-width:640px) { .header-note{display:none}.playground-shell{padding:24px 14px 22px}.playground-heading{margin-bottom:14px}.playground-heading h1{font-size:31px}.playground-heading p{font-size:12px;max-width:300px}.play-modes{width:100%;justify-content:space-between;gap:1px;padding:4px}.play-mode{padding:8px 9px;font-size:10px;gap:5px}.mode-symbol{font-size:13px}.playground-stage{min-height:720px;overflow:visible;border-radius:18px}.playground-stage .stage-canvas{height:720px;min-height:720px;border-radius:18px}.playground-controls{top:auto;bottom:14px;left:12px;right:12px;width:auto;padding:15px}.playground-controls .ctrl-group{margin-bottom:10px}.control-heading{margin-bottom:12px}.playground-controls .control-footnote{display:none}.playground-stage .hud-box{top:62px;bottom:auto;left:12px;min-width:185px;padding:9px 10px;font-size:10px}.playground-stage .scene-title{left:13px;top:14px}.scene-title strong{font-size:12px}.scene-title small{font-size:10px}.stage-export{right:12px;top:12px}.drag-tip{right:16px;bottom:443px;max-width:120px;text-align:right}.play-prompts{gap:6px;padding-top:12px}.prompt-label{width:100%;margin:0 0 2px 2px}.play-prompts button{font-size:9px;padding:7px 9px}.intro-content{width:80vw} }
</style>
