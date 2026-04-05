<template>
  <div class="home">
    <!-- Animated background -->
    <div class="bg-gradient"></div>
    <div class="bg-grid"></div>

    <!-- Top Bar -->
    <nav class="topbar">
      <div class="topbar-brand">
        <img src="@/assets/logo/Logo_Regusim.png" class="brand-logo" alt="ReguSim" />
      </div>
      <div class="topbar-links">
        <a href="https://github.com/nikmcfly/ReguSim" target="_blank" class="topbar-link">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
          GitHub
        </a>
        <span class="topbar-version">v0.1-preview</span>
      </div>
    </nav>

    <!-- Hero -->
    <section class="hero">
      <div class="hero-badge">
        <span class="badge-dot"></span>
        AI-Powered Regulatory Impact Assessment
      </div>

      <h1 class="hero-title">
        Simulate Regulation.<br>
        <span class="hero-title-accent">Predict Impact.</span>
      </h1>

      <p class="hero-subtitle">
        Upload a policy draft and ReguSim builds a multi-agent simulation of
        <strong>Indonesian financial stakeholders</strong> — from OJK and Bank Indonesia
        to fintech startups and public voices.
      </p>

      <div class="hero-scroll" @click="scrollToBottom">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="6 9 12 15 18 9"></polyline></svg>
      </div>
    </section>

    <!-- Main Content -->
    <div class="content-wrapper">
      <!-- Upload + Scenario Card -->
      <section class="launch-card">
        <div class="launch-card-inner">
          <!-- Left: Upload -->
          <div class="launch-left">
            <div class="launch-section-label">
              <span class="label-num">01</span>
              Regulatory Documents
              <span class="label-hint">PDF, MD, TXT</span>
            </div>
            <div
              class="upload-zone"
              :class="{ 'drag-over': isDragOver, 'has-files': files.length > 0 }"
              @dragover.prevent="handleDragOver"
              @dragleave.prevent="handleDragLeave"
              @drop.prevent="handleDrop"
              @click="triggerFileInput"
            >
              <input ref="fileInput" type="file" multiple accept=".pdf,.md,.txt" @change="handleFileSelect" style="display:none" :disabled="loading" />
              <div v-if="files.length === 0" class="upload-empty">
                <div class="upload-icon-ring">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
                </div>
                <div class="upload-label">Drop files or <span>browse</span></div>
              </div>
              <div v-else class="file-chips">
                <div v-for="(file, index) in files" :key="index" class="file-chip">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
                  <span class="file-chip-name">{{ file.name }}</span>
                  <button class="file-chip-remove" @click.stop="removeFile(index)">×</button>
                </div>
              </div>
            </div>
          </div>

          <!-- Right: Scenario + Button -->
          <div class="launch-right">
            <div class="launch-section-label">
              <span class="label-num">02</span>
              Regulatory Scenario
            </div>
            <div class="scenario-input-wrapper">
              <textarea
                v-model="formData.simulationRequirement"
                class="scenario-input"
                placeholder="Describe the regulation to simulate, e.g. 'OJK mandates open banking API standard for all fintech lenders by Q3 2026'"
                rows="5"
                :disabled="loading"
              ></textarea>
              <div class="engine-badge">Ollama + Neo4j · Local</div>
            </div>
            <button class="launch-btn" @click="startSimulation" :disabled="!canSubmit || loading">
              <span v-if="!loading">Launch RIA Simulation</span>
              <span v-else>Initializing...</span>
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>
            </button>
          </div>
        </div>
      </section>

      <!-- Workflow Steps -->
      <section class="workflow">
        <div class="workflow-header">
          <div class="workflow-line"></div>
          <span class="workflow-label">Workflow</span>
          <div class="workflow-line"></div>
        </div>
        <div class="steps-row">
          <div v-for="(step, i) in steps" :key="i" class="step-tile">
            <div class="step-tile-num">{{ step.num }}</div>
            <div class="step-tile-title">{{ step.title }}</div>
            <div class="step-tile-desc">{{ step.desc }}</div>
          </div>
        </div>
      </section>

      <!-- Metrics Row -->
      <section class="metrics-strip">
        <div class="metric-box">
          <div class="metric-val">100%</div>
          <div class="metric-lbl">Offline & Private</div>
        </div>
        <div class="metric-divider"></div>
        <div class="metric-box">
          <div class="metric-val">RIA</div>
          <div class="metric-lbl">Regulatory Impact Assessment</div>
        </div>
        <div class="metric-divider"></div>
        <div class="metric-box">
          <div class="metric-val">Multi-Agent</div>
          <div class="metric-lbl">LLM-driven Stakeholder Simulation</div>
        </div>
      </section>

      <!-- History -->
      <HistoryDatabase />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import HistoryDatabase from '../components/HistoryDatabase.vue'

const steps = [
  { num: '01', title: 'Regulatory Graph', desc: 'Extract entities from documents and build a regulatory knowledge graph' },
  { num: '02', title: 'Stakeholder Setup', desc: 'Generate stakeholder personas with realistic behavioral profiles' },
  { num: '03', title: 'Impact Simulation', desc: 'Multi-agent simulation of stakeholder reactions to regulation' },
  { num: '04', title: 'RIA Report', desc: 'Structured Regulatory Impact Assessment with sentiment analysis' },
  { num: '05', title: 'Stakeholder Chat', desc: 'Chat directly with any simulated stakeholder agent' },
]

const router = useRouter()
const formData = ref({ simulationRequirement: '' })
const files = ref([])
const loading = ref(false)
const error = ref('')
const isDragOver = ref(false)
const fileInput = ref(null)

const canSubmit = computed(() => {
  return formData.value.simulationRequirement.trim() !== '' && files.value.length > 0
})

const triggerFileInput = () => { if (!loading.value) fileInput.value?.click() }
const handleFileSelect = (event) => { addFiles(Array.from(event.target.files)) }
const handleDragOver = () => { isDragOver.value = true }
const handleDragLeave = () => { isDragOver.value = false }
const handleDrop = (e) => { isDragOver.value = false; addFiles(Array.from(e.dataTransfer.files)) }

const addFiles = (newFiles) => {
  const allowed = ['.pdf', '.md', '.txt']
  const valid = newFiles.filter(f => allowed.some(ext => f.name.toLowerCase().endsWith(ext)))
  files.value = [...files.value, ...valid]
}

const removeFile = (index) => { files.value.splice(index, 1) }
const scrollToBottom = () => { window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' }) }

const startSimulation = () => {
  if (!canSubmit.value || loading.value) return
  import('../store/pendingUpload.js').then(({ setPendingUpload }) => {
    setPendingUpload(files.value, formData.value.simulationRequirement)
    router.push({ name: 'Process', params: { projectId: 'new' } })
  })
}
</script>

<style scoped>
.home {
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
  background: var(--bg-base);
}

/* Background — subtle warm tone with a faint navy vignette on top */
.bg-gradient {
  position: fixed; inset: 0;
  background:
    radial-gradient(ellipse 90% 50% at 50% -5%, rgba(30,58,95,0.08) 0%, transparent 60%),
    radial-gradient(ellipse 60% 40% at 80% 10%, rgba(47,127,122,0.04) 0%, transparent 50%);
  pointer-events: none; z-index: 0;
}
.bg-grid {
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(30,58,95,0.025) 1px, transparent 1px),
    linear-gradient(90deg, rgba(30,58,95,0.025) 1px, transparent 1px);
  background-size: 60px 60px;
  pointer-events: none; z-index: 0;
}

/* TOPBAR — with subtle navy accent line */
.topbar {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 32px; height: 56px;
  background: rgba(255,255,255,0.95); backdrop-filter: blur(12px);
  border-bottom: 1px solid var(--border);
  box-shadow: 0 1px 8px rgba(30,58,95,0.04);
}
.topbar-brand { display: flex; align-items: center; }
.brand-logo {
  height: 32px; width: auto;
}
.topbar-links { display: flex; align-items: center; gap: 16px; }
.topbar-link {
  display: flex; align-items: center; gap: 6px;
  font-size: 0.8rem; font-weight: 500; color: var(--text-secondary); transition: color 0.2s;
}
.topbar-link:hover { color: var(--accent); }
.topbar-version {
  font-family: var(--font-mono); font-size: 0.68rem; color: var(--accent);
  padding: 3px 10px; border: 1px solid var(--accent-border); border-radius: var(--radius-sm);
  background: var(--accent-bg);
}

/* HERO — with decorative accent */
.hero {
  position: relative; z-index: 1; text-align: center;
  padding: 90px 32px 48px; max-width: 720px; margin: 0 auto;
}
.hero-badge {
  display: inline-flex; align-items: center; gap: 8px;
  padding: 8px 20px; border-radius: 20px;
  background: var(--accent); border: none;
  font-size: 0.74rem; font-weight: 600; color: #fff;
  margin-bottom: 32px; letter-spacing: 0.4px;
  box-shadow: 0 2px 12px rgba(30,58,95,0.18);
}
.badge-dot {
  width: 6px; height: 6px; border-radius: 50%; background: rgba(255,255,255,0.7);
  animation: pulse-dot 2.5s ease-in-out infinite;
}
@keyframes pulse-dot { 0%,100%{opacity:1} 50%{opacity:0.3} }
.hero-title {
  font-family: var(--font-serif);
  font-size: 3.4rem; font-weight: 700; line-height: 1.18;
  letter-spacing: -0.5px; margin-bottom: 24px;
  color: var(--text-primary);
}
.hero-title-accent {
  color: var(--secondary);
  -webkit-text-fill-color: var(--secondary);
}
.hero-subtitle {
  font-size: 1.05rem; line-height: 1.75; color: var(--text-secondary);
  max-width: 540px; margin: 0 auto 36px;
}
.hero-subtitle strong { color: var(--accent); font-weight: 600; }
.hero-scroll {
  display: inline-flex; align-items: center; justify-content: center;
  width: 38px; height: 38px; border-radius: 50%;
  border: 1.5px solid var(--border); color: var(--text-muted); cursor: pointer;
  transition: all 0.3s ease;
}
.hero-scroll:hover { border-color: var(--accent); color: var(--accent); background: var(--accent-bg); }

/* CONTENT */
.content-wrapper {
  position: relative; z-index: 1;
  max-width: 1060px; margin: 0 auto; padding: 0 32px 80px;
}

/* LAUNCH CARD — elevated with navy top accent */
.launch-card {
  background: var(--bg-card);
  border: 1px solid var(--border); border-radius: var(--radius-lg);
  box-shadow: 0 4px 24px rgba(30,58,95,0.08), 0 1px 4px rgba(0,0,0,0.03);
  margin-bottom: 64px;
  position: relative;
  overflow: hidden;
}

.launch-card-inner {
  display: grid; grid-template-columns: 1fr 1fr;
  overflow: hidden;
}
.launch-left, .launch-right { padding: 32px; }
.launch-left { border-right: 1px solid var(--border-light); }
.launch-section-label {
  display: flex; align-items: center; gap: 10px;
  font-size: 0.72rem; font-weight: 600; color: var(--text-muted);
  text-transform: uppercase; letter-spacing: 1.5px; margin-bottom: 18px;
}
.label-num {
  font-family: var(--font-mono); font-size: 0.68rem; color: #fff;
  background: var(--accent); padding: 3px 9px; border-radius: var(--radius-sm);
  font-weight: 700;
}
.label-hint {
  margin-left: auto; font-size: 0.66rem; color: var(--text-muted);
  font-weight: 500; text-transform: none; letter-spacing: 0;
}

/* Upload zone */
.upload-zone {
  border: 1.5px dashed var(--border); border-radius: var(--radius-md);
  min-height: 200px; display: flex; align-items: center; justify-content: center;
  cursor: pointer; transition: all 0.3s ease; background: var(--bg-warm);
}
.upload-zone:hover, .upload-zone.drag-over {
  border-color: var(--secondary); background: var(--secondary-bg);
}
.upload-zone.has-files { align-items: flex-start; padding: 16px; }
.upload-empty { text-align: center; }
.upload-icon-ring {
  width: 56px; height: 56px; border-radius: 50%;
  border: 1.5px solid var(--border); background: var(--bg-base);
  display: flex; align-items: center; justify-content: center;
  margin: 0 auto 14px; color: var(--text-muted); transition: all 0.3s ease;
}
.upload-zone:hover .upload-icon-ring {
  border-color: var(--secondary); color: var(--secondary); background: var(--secondary-bg);
}
.upload-label { font-size: 0.85rem; color: var(--text-muted); }
.upload-label span { color: var(--accent); font-weight: 600; text-decoration: underline; text-underline-offset: 2px; }

/* File chips */
.file-chips { display: flex; flex-wrap: wrap; gap: 8px; width: 100%; }
.file-chip {
  display: flex; align-items: center; gap: 8px; padding: 8px 12px;
  background: var(--bg-warm); border: 1px solid var(--border);
  border-radius: var(--radius-sm); font-size: 0.8rem; color: var(--text-secondary);
  transition: border-color 0.2s;
}
.file-chip:hover { border-color: var(--accent-border); }
.file-chip svg { color: var(--accent); flex-shrink: 0; }
.file-chip-name { flex:1; min-width:0; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; }
.file-chip-remove {
  background:none; border:none; color:var(--text-muted);
  font-size:1rem; cursor:pointer; padding:0 2px; line-height:1;
}
.file-chip-remove:hover { color: var(--rose); }

/* Scenario input */
.scenario-input-wrapper { position: relative; margin-bottom: 18px; }
.scenario-input {
  width:100%; background:var(--bg-warm); border:1px solid var(--border);
  border-radius:var(--radius-md); color:var(--text-primary);
  font-family:var(--font-sans); font-size:0.88rem; line-height:1.7;
  padding:16px; resize:vertical; outline:none; min-height:140px;
  transition:border-color 0.2s, box-shadow 0.2s;
}
.scenario-input::placeholder { color:var(--text-muted); }
.scenario-input:focus { border-color:var(--accent); box-shadow: 0 0 0 3px var(--accent-bg); }
.engine-badge {
  position:absolute; bottom:10px; right:12px;
  font-family:var(--font-mono); font-size:0.65rem; color:var(--text-muted); opacity:0.6;
}

/* Launch button — navy with teal hover */
.launch-btn {
  width:100%; display:flex; align-items:center; justify-content:center; gap:10px;
  padding:14px 24px; border:none; border-radius:var(--radius-md);
  background:var(--accent); color:#fff;
  font-weight:600; font-size:0.92rem; cursor:pointer;
  transition:all 0.25s ease; letter-spacing: 0.2px;
}
.launch-btn:hover:not(:disabled) {
  background: var(--secondary); transform:translateY(-1px);
  box-shadow: 0 4px 16px rgba(47,127,122,0.25);
}
.launch-btn:disabled { opacity:0.3; cursor:not-allowed; transform:none; }

/* WORKFLOW */
.workflow { margin-bottom: 56px; }
.workflow-header { display:flex; align-items:center; gap:16px; margin-bottom:28px; }
.workflow-line { flex:1; height:1px; background:var(--border); }
.workflow-label {
  font-size:0.68rem; font-weight:600; text-transform:uppercase;
  letter-spacing:2px; color:var(--text-muted);
}
.steps-row { display:grid; grid-template-columns:repeat(5,1fr); gap:14px; }
.step-tile {
  background:var(--bg-card); border:1px solid var(--border);
  border-radius:var(--radius-md); padding:22px 18px;
  transition:all 0.3s ease;
  position: relative;
  border-top: 2px solid var(--border);
}
.step-tile:hover {
  border-color: var(--accent-border);
  border-top-color: var(--accent);
  box-shadow: 0 4px 16px rgba(30,58,95,0.08);
  transform: translateY(-3px);
}
.step-tile-num {
  font-family:var(--font-mono); font-size:0.72rem; font-weight:700;
  color:#fff; background: var(--accent);
  display: inline-block; padding: 2px 8px; border-radius: var(--radius-sm);
  margin-bottom:12px;
}
.step-tile-title {
  font-family:var(--font-serif); font-size:0.92rem; font-weight:600;
  margin-bottom:8px; color: var(--text-primary);
}
.step-tile-desc { font-size:0.76rem; line-height:1.55; color:var(--text-muted); }

/* METRICS — navy background for contrast */
.metrics-strip {
  display:flex; align-items:center; justify-content:center;
  background:var(--accent); border:none;
  border-radius:var(--radius-lg); padding:32px 0; margin-bottom:56px;
  box-shadow: 0 4px 20px rgba(30,58,95,0.2);
}
.metric-box { flex:1; text-align:center; padding:0 24px; }
.metric-val {
  font-family:var(--font-serif); font-size:1.5rem; font-weight:700;
  margin-bottom:6px; color: #fff;
}
.metric-lbl { font-size:0.76rem; color: rgba(255,255,255,0.7); }
.metric-divider { width:1px; height:44px; background:rgba(255,255,255,0.2); }

@media (max-width: 900px) {
  .launch-card-inner { grid-template-columns: 1fr; }
  .launch-left { border-right:none; border-bottom:1px solid var(--border); }
  .steps-row { grid-template-columns: 1fr 1fr; }
  .hero-title { font-size: 2.4rem; }
}
</style>
