<template>
  <div :class="['app-shell', { projector: projectorMode }]">
    <main v-if="activeView === 'intro'" class="intro-page">
      <canvas ref="introCanvas" class="intro-art" @pointermove="moveIntroArt" @pointerdown="moveIntroArt"></canvas>
      <div class="intro-topline"><span class="intro-mark">∫</span><span>ÉTS Montréal <i>·</i> MAT145</span><span class="intro-live"><b></b> Une expérience interactive</span></div>
      <div class="intro-content">
        <div class="intro-kicker">CALCUL DIFFÉRENTIEL ET INTÉGRAL</div>
        <h1>Les mathématiques<br/>prennent <em>forme.</em></h1>
        <p>Explorez les idées derrière le calcul, une courbe à la fois.</p>
        <button class="intro-enter" @click="switchView('landing')">Entrer dans le studio <span>↗</span></button>
        <div class="intro-hint"><span class="hint-line"></span>Déplacez le pointeur pour explorer</div>
      </div>
      <div class="intro-equation equation-one">∫<sub>a</sub><sup>b</sup> f(x) dx</div>
      <div class="intro-equation equation-two">lim<sub>h→0</sub> &nbsp; Δy / Δx</div>
      <div class="intro-index"><span>01</span><span class="index-rule"></span><span>04</span><span>CONCEPTS À EXPLORER</span></div>
      <div class="intro-bottom"><span>Un espace pour voir, toucher et comprendre.</span><span>MAT145 STUDIO <b>©</b> 2026</span></div>
    </main>
    <header v-if="activeView !== 'intro'" class="navbar">
      <div class="nav-inner">
        <div class="brand" @click="switchView('landing')">
          <div class="ets-logo">ÉTS</div>
          <div>
            <div class="brand-title">MAT145 Studio <span class="badge">ÉTS Montréal</span></div>
            <div class="brand-sub">Calcul différentiel et intégral • Laboratoire visuel interactif</div>
          </div>
        </div>

        <nav class="nav-tabs">
          <button :class="['nav-btn', { active: activeView === 'landing' }]" @click="switchView('landing')">Accueil</button>
          <button :class="['nav-btn', { active: activeView === 'studio' }]" @click="switchView('studio')">Explorer les laboratoires</button>
          <button :class="['nav-btn', { active: activeView === 'presets' }]" @click="switchView('presets')">Scénarios</button>
        </nav>

        <button :class="['btn-outline', { active: projectorMode }]" @click="projectorMode = !projectorMode">
          Mode présentation
        </button>
      </div>
    </header>

    <!-- VUE 1 : LANDING PAGE AVEC ANIMATIONS TEMPS RÉEL -->
    <main v-if="activeView === 'landing'" class="main-content">
      <section class="hero-section">
        <canvas ref="heroWaveCanvas" class="hero-bg-canvas"></canvas>
        <div class="hero-grid">
          <div class="hero-copy">
            <div class="hero-pill">
              <span class="dot"></span>
              <span>ÉTS MONTRÉAL • DÉPARTEMENT DES ENSEIGNEMENTS GÉNÉRAUX</span>
            </div>
            <h1 class="hero-h1">
              Donnez vie aux concepts de <span class="grad-text">MAT145 en Temps Réel.</span>
            </h1>
            <p class="hero-p">
              Une plateforme web moderne conçue pour les professeurs, chargés de cours et étudiants de l'<strong>École de technologie supérieure</strong>. Illustrez les limites différentielles, les sommes de Riemann, les solides de révolution 3D (disques et rondelles) et la convergence des séries de Taylor à 60 FPS.
            </p>
            <div class="hero-actions">
              <button class="btn-primary" @click="openLab('derivative')">Explorer le studio</button>
              <button class="btn-secondary" @click="switchView('presets')">Voir les scénarios</button>
            </div>
            <div class="metrics-row">
              <div><strong>60 FPS</strong><span>Moteur Temps Réel</span></div>
              <div><strong>4 Labs</strong><span>Syllabus MAT145</span></div>
              <div><strong>2D & 3D</strong><span>Disques & Rondelles</span></div>
              <div><strong>100%</strong><span>Compatible Moodle</span></div>
            </div>
          </div>

          <div class="hero-card">
            <div class="card-header">
              <span>● SIMULATION EN DIRECT : ONDE & TANGENTE</span>
              <span class="status-ok">60 FPS Actif</span>
            </div>
            <canvas ref="heroMiniCanvas" class="mini-canvas"></canvas>
            <div class="card-footer">
              <span>f(x, t) = sin(x - ωt) · cos(0.4x)</span>
              <span style="color:#c18d41">Riemann & Tangente</span>
            </div>
          </div>
        </div>
      </section>

      <section class="bento-section">
        <div class="sec-head">
          <span class="eyebrow">PROGRAMME OFFICIEL MAT145 — ÉTS MONTRÉAL</span>
          <h2>Les 4 Laboratoires Visuels du Cours</h2>
          <p>Cliquez sur un module pour lancer l'environnement de simulation mathématique.</p>
        </div>
        <div class="bento-grid">
          <div v-for="mod in bentoCards" :key="mod.id" class="bento-card" @click="openLab(mod.id)">
            <div>
              <div class="bento-meta">
                <span class="mod-badge">{{ mod.badge }}</span>
                <span class="mod-weeks">{{ mod.weeks }}</span>
              </div>
              <h3>{{ mod.title }}</h3>
              <p>{{ mod.desc }}</p>
              <div class="formula-box">{{ mod.formula }}</div>
            </div>
            <div class="bento-link"><span>Lancer ce laboratoire</span><span>→</span></div>
          </div>
        </div>
      </section>
    </main>

    <!-- VUE 2 : STUDIO INTERACTIF -->
    <main v-else-if="activeView === 'studio'" class="studio-container">
      <div class="studio-bar">
        <div class="lab-buttons">
          <button v-for="m in bentoCards" :key="m.id" :class="['lab-tab', { active: activeLab === m.id }]" @click="selectLab(m.id)">
            {{ m.shortTitle }}
          </button>
        </div>
        <button class="btn-outline" @click="exportPNG">Exporter en PNG</button>
      </div>

      <div class="studio-grid">
        <div class="stage-card">
          <canvas ref="labCanvas" class="stage-canvas" @mousedown="onCanvasDown" @mousemove="onCanvasMove" @mouseup="onCanvasUp"></canvas>
          <div class="hud-box">
            <div v-for="(line, i) in hudLines" :key="i" class="hud-line">
              <span>{{ line.label }}</span>
              <strong :style="{ color: line.color }">{{ line.value }}</strong>
            </div>
          </div>
        </div>

        <div class="controls-card">
          <h3>Paramètres en direct</h3>

          <template v-if="activeLab === 'derivative'">
            <div class="ctrl-group">
              <label>Fonction f(x)</label>
              <select v-model="d1.fn" @change="drawLab">
                <option value="parabola">f(x) = x² - 2x (Parabole)</option>
                <option value="cubic">f(x) = 0.25x³ - 0.75x + 0.5 (Cubique)</option>
                <option value="trig">f(x) = 1.8 sin(x) + 0.4 cos(2x)</option>
              </select>
            </div>
            <div class="ctrl-group">
              <label>Point de tangence a : <strong>{{ d1.a.toFixed(2) }}</strong></label>
              <input type="range" v-model.number="d1.a" min="-3" max="3" step="0.02" @input="drawLab" />
            </div>
            <div class="ctrl-group">
              <label>Incrément Δx (h) : <strong>{{ d1.h.toFixed(3) }}</strong></label>
              <input type="range" v-model.number="d1.h" min="0.005" max="2.4" step="0.005" @input="drawLab" />
            </div>
            <button class="btn-primary full" @click="animateHToZero">▶ Animer la limite h → 0</button>
          </template>

          <template v-else-if="activeLab === 'riemann'">
            <div class="ctrl-group">
              <label>Méthode d'intégration</label>
              <select v-model="d2.method" @change="drawLab">
                <option value="midpoint">Point Milieu (Erreur O(1/n²))</option>
                <option value="trapezoid">Trapèzes (Erreur O(1/n²))</option>
                <option value="left">Somme à Gauche (Erreur O(1/n))</option>
                <option value="right">Somme à Droite (Erreur O(1/n))</option>
              </select>
            </div>
            <div class="ctrl-group">
              <label>Nombre de subdivisions n : <strong>{{ d2.n }}</strong></label>
              <input type="range" v-model.number="d2.n" min="2" max="90" step="1" @input="drawLab" />
            </div>
            <div class="ctrl-group">
              <label>Borne supérieure b : <strong>{{ d2.b.toFixed(2) }}</strong></label>
              <input type="range" v-model.number="d2.b" min="1.5" max="5.5" step="0.25" @input="drawLab" />
            </div>
          </template>

          <template v-else-if="activeLab === 'revolution'">
            <div class="ctrl-group">
              <label>Profil 3D (Disques & Rondelles)</label>
              <select v-model="d3.preset" @change="drawLab">
                <option value="paraboloid">Paraboloïde plein : R(x) = √x</option>
                <option value="washer">Rondelle creuse : R(x) = √x + 0.6, r(x) = 0.25x</option>
                <option value="vase">Tuyère Venturi : R(x) = 1.3 + 0.55 cos(1.5x)</option>
              </select>
            </div>
            <div class="ctrl-group">
              <label>Angle de révolution θ : <strong>{{ d3.theta }}°</strong></label>
              <input type="range" v-model.number="d3.theta" min="15" max="360" step="5" @input="drawLab" />
            </div>
            <div class="ctrl-group">
              <label>Position de la rondelle x_k : <strong>{{ d3.sliceX.toFixed(2) }}</strong></label>
              <input type="range" v-model.number="d3.sliceX" min="0.3" max="3.4" step="0.05" @input="drawLab" />
            </div>
            <p class="hint">💡 Cliquez et glissez sur la scène pour faire pivoter la caméra 3D.</p>
          </template>

          <template v-else>
            <div class="ctrl-group">
              <label>Fonction analytique f(x)</label>
              <select v-model="d4.fn" @change="drawLab">
                <option value="sin">f(x) = sin(x) (R = ∞)</option>
                <option value="cos">f(x) = cos(x) (R = ∞)</option>
                <option value="exp">f(x) = e^x (R = ∞)</option>
                <option value="geom">f(x) = 1 / (1 - x) (R = 1, Série géométrique)</option>
                <option value="ln">f(x) = ln(1 + x) (R = 1)</option>
              </select>
            </div>
            <div class="ctrl-group">
              <label>Degré du polynôme de Taylor n : <strong>{{ d4.n }}</strong></label>
              <input type="range" v-model.number="d4.n" min="0" max="15" step="1" @input="drawLab" />
            </div>
          </template>
        </div>
      </div>
    </main>

    <!-- VUE 3 : SCÉNARIOS PRÉCONFIGURÉS -->
    <main v-else class="bento-section">
      <div class="sec-head">
        <span class="eyebrow">EXPOSÉS MAGISTRAUX ÉTS</span>
        <h2>Scénarios Pédagogiques en 1 Clic</h2>
      </div>
      <div class="bento-grid">
        <div v-for="sc in scenarios" :key="sc.id" class="bento-card" @click="runScenario(sc.id)">
          <div>
            <span class="mod-badge">{{ sc.tag }}</span>
            <h3 style="margin-top:12px">{{ sc.title }}</h3>
            <p>{{ sc.desc }}</p>
          </div>
          <div class="bento-link"><span>Charger ce scénario</span><span>→</span></div>
        </div>
      </div>
    </main>

    <footer class="footer">
      MAT145 — Calcul différentiel et intégral • École de technologie supérieure (ÉTS Montréal)
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, nextTick } from 'vue';

const activeView = ref('intro');
const activeLab = ref('derivative');
const projectorMode = ref(false);

const introCanvas = ref(null);
const introPointer = reactive({ x: 0.72, y: 0.45 });
const heroWaveCanvas = ref(null);
const heroMiniCanvas = ref(null);
const labCanvas = ref(null);
const hudLines = ref([]);

let animFrame = null;
let heroTime = 0;

const d1 = reactive({ fn: 'parabola', a: 1.0, h: 1.2 });
const d2 = reactive({ method: 'midpoint', n: 10, b: 4.0 });
const d3 = reactive({ preset: 'washer', theta: 260, sliceX: 2.0, yaw: 0.72, pitch: 0.35, dragging: false, lx: 0, ly: 0 });
const d4 = reactive({ fn: 'sin', n: 5 });

const bentoCards = [
  { id: 'derivative', shortTitle: '01. Dérivée & Tangente', badge: 'LAB 01 • CALCUL DIFFÉRENTIEL', weeks: 'Semaines 1 à 4', title: 'Taux de Variation, Sécante & Linéarisation', desc: "Faites tendre l'incrément Δx (h → 0) pour observer la corde sécante pivoter vers la tangente exacte.", formula: "f'(a) = lim_{h → 0} [ f(a + h) - f(a) ] / h" },
  { id: 'riemann', shortTitle: '02. Sommes de Riemann', badge: 'LAB 02 • CALCUL INTÉGRAL', weeks: 'Semaines 5 à 8', title: 'Sommes de Riemann & Théorème Fondamental', desc: 'Comparez la vitesse de convergence entre les sommes à gauche, à droite, au point milieu et par trapèzes.', formula: '∫_a^b f(x) dx = lim_{n → ∞} Σ f(x_i*) · Δx' },
  { id: 'revolution', shortTitle: '03. Solides 3D', badge: 'LAB 03 • GÉOMÉTRIE 3D', weeks: 'Semaines 9 à 11', title: 'Solides de Révolution 3D (Disques & Rondelles)', desc: 'Moteur 3D interactif avec rayon externe R(x), interne r(x) et découpe de rondelle élémentaire dV.', formula: 'V = π ∫_a^b ( [R(x)]² - [r(x)]² ) dx' },
  { id: 'taylor', shortTitle: '04. Séries de Taylor', badge: 'LAB 04 • SUITES ET SÉRIES', weeks: 'Semaines 12 à 13', title: 'Séries de Taylor & Intervalle de Convergence', desc: "Superposez les polynômes de Taylor P_n(x) jusqu'à l'ordre 15 et visualisez le rayon de convergence R.", formula: 'P_n(x) = Σ_{k=0}^n [ f^(k)(0) / k! ] · x^k' }
];

const scenarios = [
  { id: 1, tag: 'SCÉNARIO 01 • DÉRIVÉE', title: "Tangente horizontale et point d'inflexion", desc: 'Polynôme cubique 0.25x³ - 0.75x + 0.5 au voisinage de son extremum local a = 1.0.' },
  { id: 2, tag: 'SCÉNARIO 02 • INTÉGRATION', title: 'Précision quadratique du Point Milieu', desc: 'Intégration avec seulement n = 6 subdivisions illustrant la compensation des erreurs.' },
  { id: 3, tag: 'SCÉNARIO 03 • SOLIDES 3D', title: 'Coupe à 240° d’une Rondelle Creuse', desc: 'Solide 3D ouvert à θ = 240° montrant le rayon externe R(x) et le trou conique interne r(x).' },
  { id: 4, tag: 'SCÉNARIO 04 • TAYLOR', title: 'Mur de divergence de la série géométrique', desc: 'Série 1/(1-x) à l’ordre n = 11 montrant la frontière stricte de convergence en x = ±1.' }
];

function switchView(v) {
  activeView.value = v;
  if (v === 'studio') nextTick(() => drawLab());
}

function moveIntroArt(event) {
  const rect = event.currentTarget.getBoundingClientRect();
  introPointer.x = (event.clientX - rect.left) / rect.width;
  introPointer.y = (event.clientY - rect.top) / rect.height;
}

function openLab(id) {
  activeLab.value = id;
  activeView.value = 'studio';
  nextTick(() => drawLab());
}

function selectLab(id) {
  activeLab.value = id;
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
  if (activeView.value === 'intro' && art) {
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
  if (activeView.value === 'landing') {
    const bg = heroWaveCanvas.value;
    if (bg) {
      if (bg.width !== bg.clientWidth) { bg.width = bg.clientWidth; bg.height = bg.clientHeight; }
      const ctx = bg.getContext('2d');
      ctx.clearRect(0, 0, bg.width, bg.height);
      for (let i = 0; i < 5; i++) {
        ctx.beginPath();
        ctx.strokeStyle = i % 2 === 0 ? 'rgba(56, 189, 248, 0.16)' : 'rgba(239, 62, 66, 0.14)';
        ctx.lineWidth = 1.5;
        for (let x = 0; x <= bg.width; x += 12) {
          const y = bg.height * 0.5 + Math.sin(x * 0.004 + heroTime + i * 0.7) * (42 + i * 14);
          if (x === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
        }
        ctx.stroke();
      }
    }
    const mini = heroMiniCanvas.value;
    if (mini) {
      if (mini.width !== mini.clientWidth) { mini.width = mini.clientWidth; mini.height = mini.clientHeight; }
      const ctx = mini.getContext('2d');
      const w = mini.width, h = mini.height;
      ctx.fillStyle = '#f1f2f8';
      ctx.fillRect(0, 0, w, h);
      const midY = h / 2, sx = w / 10, sy = h / 4.2;
      const nR = 14, rw = w / nR;
      for (let i = 0; i < nR; i++) {
        const mx = ((i + 0.5) * rw - w / 2) / sx;
        const val = Math.sin(mx - heroTime * 1.2) * Math.cos(0.4 * mx);
        const ry = midY - val * sy;
        ctx.fillStyle = 'rgba(116, 131, 203, 0.18)';
        ctx.fillRect(i * rw + 2, Math.min(midY, ry), rw - 4, Math.abs(midY - ry));
      }
      ctx.strokeStyle = '#7483cb';
      ctx.lineWidth = 3;
      ctx.beginPath();
      for (let px = 0; px <= w; px += 3) {
        const mx = (px - w / 2) / sx;
        const py = midY - Math.sin(mx - heroTime * 1.2) * Math.cos(0.4 * mx) * sy;
        if (px === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
      }
      ctx.stroke();
    }
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
  if (activeLab.value === 'revolution') { d3.dragging = true; d3.lx = e.clientX; d3.ly = e.clientY; }
}
function onCanvasMove(e) {
  if (activeLab.value === 'revolution' && d3.dragging) {
    d3.yaw += (e.clientX - d3.lx) * 0.008;
    d3.pitch = Math.max(-1.2, Math.min(1.2, d3.pitch - (e.clientY - d3.ly) * 0.008));
    d3.lx = e.clientX; d3.ly = e.clientY;
    drawLab();
  }
}
function onCanvasUp() { d3.dragging = false; }

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
.intro-page { min-height:100svh; position:relative; overflow:hidden; isolation:isolate; background:radial-gradient(ellipse at 74% 46%,#f0f1fb 0%,#f6f6f8 42%,#f5f5f7 76%); }
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
</style>
