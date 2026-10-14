<template>
  <div :class="{ 'projector-mode': projectorMode }" class="min-h-screen flex flex-col bg-[#050811] text-slate-100">
    <header class="sticky top-0 z-50 bg-[#050811]/85 backdrop-blur-xl border-b border-slate-800 px-6 py-4">
      <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-4">
        <div class="flex items-center gap-3 cursor-pointer" @click="activeView = 'landing">
          <span class="bg-[#EF3E42] text-white font-black px-3 py-1.5 rounded-lg">éTS</span>
          <div>
            <h1 class="font-extrabold text-white">MAT145 Studio v2.0</h1>
            <p class="text-xs text-slate-400">Calcul différentiel et intégral • éTS Montréal</p>
            </div>
            </div>
            <nav class="flex gap-2 bg-slate-900 p-1 rounded-xl border border-slate-800">
              <button @click="activeView = 'landing" :class="activeView === 'landing' ? 'bg-[#EF3E42] text-white' : 'text-slate-400" class="px-4 py-1.5 rounded-lg text-sm font-semibold">Accueil</button>
              <button @click="activeView = 'studio" :class="activeView === 'studio' ? 'bg-[#EF3E42] text-white' : 'text-slate-400" class="px-4 py-1.5 rounded-lg text-sm font-semibold">Studio Interactif (4 Labs)</button>
              </nav>
              <button @click="projectorMode = !projectorMode" class="px-3.5 py-1.5 rounded-lg border border-slate-700 text-xs font-semibold text-slate-200">
                📽️ Mode Amphithéâtre
                </button>
                </div>
                </header>
                <main class="flex-1 max-w-7xl mx-auto w-full p-6">
                  <section v-if="activeView === 'landing" class="py-16 space-y-6">
                    <div class="inline-flex items-center gap-2 px-3.5 py-1.5 rounded-full bg-slate-900 border border-red-500/30 text-xs text-red-300 font-semibold">
                      ● éCOLE DE TECHNOLOGIE SUPéRIEURE • DéPARTEMENT DES ENSEIGNEMENTS GéNéRAUX
                      </div>
                      <h2 class="text-5xl font-black text-white leading-tight">
                        Visualisez le Calcul Différentiel et Intégral en Temps Réel.
                        </h2>
                        <p class="text-lg text-slate-300 max-w-2xl">
                          Plateforme interactive à 60 FPS pour le cours MAT145 : sécante et tangente dynamique (h → 0), sommes de Riemann, solides de révolution 3D (disques et rondelles) et séries de Taylor.
                          </p>
                          <button @click="activeView = 'studio" class="px-6 py-3.5 rounded-xl bg-[#EF3E42] text-white font-bold">
                            🚀 Ouvrir le Studio Interactif
                            </button>
                            </section>
                            <section v-else class="space-y-6">
                              <div class="flex gap-2">
                                <button v-for="m in modules" :key="m.id" @click="activeModule = m.id" :class="activeModule === m.id ? 'border-sky-400 bg-sky-500/15 text-sky-400' : 'border-slate-800 bg-slate-900 text-slate-400" class="px-4 py-2 rounded-xl border text-sm font-semibold">
                                  {{ m.title }}
                                  </button>
                                  </div>
                                  </section>
                                  </main>
                                  </div>
                                  </template>

                                  <script setup>
                                    import { ref } from 'vue';
                                    const activeView = ref('landing');
                                    const activeModule = ref('derivative');
                                    const projectorMode = ref(false);
                                    const modules = [
                                      { id: 'derivative', title: '01. Dérive & Tangente (h → 0)' },
                                      { id: 'riemann', title: '02. Sommes de Riemann' },
                                      { id: 'revolution', title: '03. Solides de Révolution 3D' },
                                      { id: 'taylor', title: '04. Séries de Taylor' },
                                      ];
</script>
