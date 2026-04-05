<template>
  <div 
    class="history-database"
    :class="{ 'no-projects': projects.length === 0 && !loading }"
    ref="historyContainer"
  >
    <!-- Background decoration: technical grid lines (only shown when there are projects) -->
    <div v-if="projects.length > 0 || loading" class="tech-grid-bg">
      <div class="grid-pattern"></div>
      <div class="gradient-overlay"></div>
    </div>

    <!-- Title section -->
    <div class="section-header">
      <div class="section-line"></div>
      <span class="section-title">Simulation Records</span>
      <div class="section-line"></div>
    </div>

    <!-- Card container (only shown when there are projects) -->
    <div v-if="projects.length > 0" class="cards-container" :class="{ expanded: isExpanded }" :style="containerStyle">
      <div 
        v-for="(project, index) in projects" 
        :key="project.simulation_id"
        class="project-card"
        :class="{ expanded: isExpanded, hovering: hoveringCard === index }"
        :style="getCardStyle(index)"
        @mouseenter="hoveringCard = index"
        @mouseleave="hoveringCard = null"
        @click="navigateToProject(project)"
      >
        <!-- Card header: simulation_id and feature availability status -->
        <div class="card-header">
          <span class="card-id">{{ formatSimulationId(project.simulation_id) }}</span>
          <div class="card-status-icons">
            <span
              class="status-icon"
              :class="{ available: project.project_id, unavailable: !project.project_id }"
              title="Graph Construction"
            >◇</span>
            <span
              class="status-icon available"
              title="Environment Setup"
            >◈</span>
            <span
              class="status-icon"
              :class="{ available: project.report_id, unavailable: !project.report_id }"
              title="Analysis Report"
            >◆</span>
          </div>
        </div>

        <!-- File list area -->
        <div class="card-files-wrapper">
          <!-- Corner decoration - viewfinder style -->
          <div class="corner-mark top-left-only"></div>

          <!-- File list -->
          <div class="files-list" v-if="project.files && project.files.length > 0">
            <div 
              v-for="(file, fileIndex) in project.files.slice(0, 3)" 
              :key="fileIndex"
              class="file-item"
            >
              <span class="file-tag" :class="getFileType(file.filename)">{{ getFileTypeLabel(file.filename) }}</span>
              <span class="file-name">{{ truncateFilename(file.filename, 20) }}</span>
            </div>
            <!-- If there are more files, show indicator -->
            <div v-if="project.files.length > 3" class="files-more">
              +{{ project.files.length - 3 }} files
            </div>
          </div>
          <!-- Placeholder when no files -->
          <div class="files-empty" v-else>
            <span class="empty-file-icon">◇</span>
            <span class="empty-file-text">No Files</span>
          </div>
        </div>

        <!-- Card title (first 20 characters of simulation requirement used as title) -->
        <h3 class="card-title">{{ getSimulationTitle(project.simulation_requirement) }}</h3>

        <!-- Card description (full simulation requirement display) -->
        <p class="card-desc">{{ truncateText(project.simulation_requirement, 55) }}</p>

        <!-- Card footer -->
        <div class="card-footer">
          <div class="card-datetime">
            <span class="card-date">{{ formatDate(project.created_at) }}</span>
            <span class="card-time">{{ formatTime(project.created_at) }}</span>
          </div>
          <span class="card-progress" :class="getProgressClass(project)">
            <span class="status-dot">●</span> {{ formatRounds(project) }}
          </span>
        </div>

        <!-- Bottom decoration line (expands on hover) -->
        <div class="card-bottom-line"></div>
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="loading" class="loading-state">
      <span class="loading-spinner"></span>
      <span class="loading-text">Loading...</span>
    </div>

    <!-- Simulation playback details modal -->
    <Teleport to="body">
      <Transition name="modal">
        <div v-if="selectedProject" class="modal-overlay" @click.self="closeModal">
          <div class="modal-content">
            <!-- Modal header -->
            <div class="modal-header">
              <div class="modal-title-section">
                <span class="modal-id">{{ formatSimulationId(selectedProject.simulation_id) }}</span>
                <span class="modal-progress" :class="getProgressClass(selectedProject)">
                  <span class="status-dot">●</span> {{ formatRounds(selectedProject) }}
                </span>
                <span class="modal-create-time">{{ formatDate(selectedProject.created_at) }} {{ formatTime(selectedProject.created_at) }}</span>
              </div>
              <button class="modal-close" @click="closeModal">×</button>
            </div>

            <!-- Modal content -->
            <div class="modal-body">
              <!-- Simulation requirement -->
              <div class="modal-section">
                <div class="modal-label">Simulation Requirement</div>
                <div class="modal-requirement">{{ selectedProject.simulation_requirement || 'None' }}</div>
              </div>

              <!-- File list -->
              <div class="modal-section">
                <div class="modal-label">Associated Files</div>
                <div class="modal-files" v-if="selectedProject.files && selectedProject.files.length > 0">
                  <div v-for="(file, index) in selectedProject.files" :key="index" class="modal-file-item">
                    <span class="file-tag" :class="getFileType(file.filename)">{{ getFileTypeLabel(file.filename) }}</span>
                    <span class="modal-file-name">{{ file.filename }}</span>
                  </div>
                </div>
                <div class="modal-empty" v-else>No Associated Files</div>
              </div>
            </div>

            <!-- Simulation playback divider -->
            <div class="modal-divider">
              <span class="divider-line"></span>
              <span class="divider-text">Simulation Playback</span>
              <span class="divider-line"></span>
            </div>

            <!-- Navigation buttons -->
            <div class="modal-actions">
              <button
                class="modal-btn btn-project"
                @click="goToProject"
                :disabled="!selectedProject.project_id"
              >
                <span class="btn-step">Step1</span>
                <span class="btn-icon">◇</span>
                <span class="btn-text">Graph Construction</span>
              </button>
              <button
                class="modal-btn btn-simulation"
                @click="goToSimulation"
              >
                <span class="btn-step">Step2</span>
                <span class="btn-icon">◈</span>
                <span class="btn-text">Environment Setup</span>
              </button>
              <button
                class="modal-btn btn-report"
                @click="goToReport"
                :disabled="!selectedProject.report_id"
              >
                <span class="btn-step">Step4</span>
                <span class="btn-icon">◆</span>
                <span class="btn-text">Analysis Report</span>
              </button>
            </div>
            <!-- Playback unavailable notice -->
            <div class="modal-playback-hint">
              <span class="hint-text">Step3 "Start Simulation" and Step5 "Deep Interaction" must be launched during execution and do not support history playback</span>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, onActivated, watch, nextTick } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { getSimulationHistory } from '../api/simulation'

const router = useRouter()
const route = useRoute()

// State
const projects = ref([])
const loading = ref(true)
const isExpanded = ref(false)
const hoveringCard = ref(null)
const historyContainer = ref(null)
const selectedProject = ref(null)  // Currently selected project (for modal)
let observer = null
let isAnimating = false  // Animation lock to prevent flickering
let expandDebounceTimer = null  // Debounce timer
let pendingState = null  // Records the target state to be executed

// Card layout configuration - adjusted to wider aspect ratio
const CARDS_PER_ROW = 4
const CARD_WIDTH = 280
const CARD_HEIGHT = 280
const CARD_GAP = 24

// Dynamically calculate container height style
const containerStyle = computed(() => {
  if (!isExpanded.value) {
    // Collapsed state: fixed height
    return { minHeight: '420px' }
  }

  // Expanded state: dynamically calculate height based on card count
  const total = projects.value.length
  if (total === 0) {
    return { minHeight: '280px' }
  }

  const rows = Math.ceil(total / CARDS_PER_ROW)
  // Calculate actual required height: rows * card height + (rows-1) * gap + small bottom spacing
  const expandedHeight = rows * CARD_HEIGHT + (rows - 1) * CARD_GAP + 10

  return { minHeight: `${expandedHeight}px` }
})

// Get card style
const getCardStyle = (index) => {
  const total = projects.value.length

  if (isExpanded.value) {
    // Expanded state: grid layout
    const transition = 'transform 700ms cubic-bezier(0.23, 1, 0.32, 1), opacity 700ms cubic-bezier(0.23, 1, 0.32, 1), box-shadow 0.3s ease, border-color 0.3s ease'

    const col = index % CARDS_PER_ROW
    const row = Math.floor(index / CARDS_PER_ROW)

    // Calculate card count for current row, ensure each row is centered
    const currentRowStart = row * CARDS_PER_ROW
    const currentRowCards = Math.min(CARDS_PER_ROW, total - currentRowStart)

    const rowWidth = currentRowCards * CARD_WIDTH + (currentRowCards - 1) * CARD_GAP

    const startX = -(rowWidth / 2) + (CARD_WIDTH / 2)
    const colInRow = index % CARDS_PER_ROW
    const x = startX + colInRow * (CARD_WIDTH + CARD_GAP)

    // Expand downward, increase spacing from title
    const y = 20 + row * (CARD_HEIGHT + CARD_GAP)

    return {
      transform: `translate(${x}px, ${y}px) rotate(0deg) scale(1)`,
      zIndex: 100 + index,
      opacity: 1,
      transition: transition
    }
  } else {
    // Collapsed state: fan-shaped stacking
    const transition = 'transform 700ms cubic-bezier(0.23, 1, 0.32, 1), opacity 700ms cubic-bezier(0.23, 1, 0.32, 1), box-shadow 0.3s ease, border-color 0.3s ease'

    const centerIndex = (total - 1) / 2
    const offset = index - centerIndex

    const x = offset * 35
    // Adjust starting position, close to title but maintain proper spacing
    const y = 25 + Math.abs(offset) * 8
    const r = offset * 3
    const s = 0.95 - Math.abs(offset) * 0.05

    return {
      transform: `translate(${x}px, ${y}px) rotate(${r}deg) scale(${s})`,
      zIndex: 10 + index,
      opacity: 1,
      transition: transition
    }
  }
}

// Get style class based on round progress
const getProgressClass = (simulation) => {
  const current = simulation.current_round || 0
  const total = simulation.total_rounds || 0

  if (total === 0 || current === 0) {
    // Not started
    return 'not-started'
  } else if (current >= total) {
    // Completed
    return 'completed'
  } else {
    // In progress
    return 'in-progress'
  }
}

// Format date (only display date part)
const formatDate = (dateStr) => {
  if (!dateStr) return ''
  try {
    const date = new Date(dateStr)
    return date.toISOString().slice(0, 10)
  } catch {
    return dateStr?.slice(0, 10) || ''
  }
}

// Format time (display hours:minutes)
const formatTime = (dateStr) => {
  if (!dateStr) return ''
  try {
    const date = new Date(dateStr)
    const hours = date.getHours().toString().padStart(2, '0')
    const minutes = date.getMinutes().toString().padStart(2, '0')
    return `${hours}:${minutes}`
  } catch {
    return ''
  }
}

// Truncate text
const truncateText = (text, maxLength) => {
  if (!text) return ''
  return text.length > maxLength ? text.slice(0, maxLength) + '...' : text
}

// Generate title from simulation requirement (first 20 characters)
const getSimulationTitle = (requirement) => {
  if (!requirement) return 'Unnamed Simulation'
  const title = requirement.slice(0, 20)
  return requirement.length > 20 ? title + '...' : title
}

// Format simulation_id display (first 6 characters)
const formatSimulationId = (simulationId) => {
  if (!simulationId) return 'SIM_UNKNOWN'
  const prefix = simulationId.replace('sim_', '').slice(0, 6)
  return `SIM_${prefix.toUpperCase()}`
}

// Format round display (current round/total rounds)
const formatRounds = (simulation) => {
  const current = simulation.current_round || 0
  const total = simulation.total_rounds || 0
  if (total === 0) return 'Not Started'
  return `${current}/${total} rounds`
}

// Get file type (for styling)
const getFileType = (filename) => {
  if (!filename) return 'other'
  const ext = filename.split('.').pop()?.toLowerCase()
  const typeMap = {
    'pdf': 'pdf',
    'doc': 'doc', 'docx': 'doc',
    'xls': 'xls', 'xlsx': 'xls', 'csv': 'xls',
    'ppt': 'ppt', 'pptx': 'ppt',
    'txt': 'txt', 'md': 'txt', 'json': 'code',
    'jpg': 'img', 'jpeg': 'img', 'png': 'img', 'gif': 'img',
    'zip': 'zip', 'rar': 'zip', '7z': 'zip'
  }
  return typeMap[ext] || 'other'
}

// Get file type label text
const getFileTypeLabel = (filename) => {
  if (!filename) return 'FILE'
  const ext = filename.split('.').pop()?.toUpperCase()
  return ext || 'FILE'
}

// Truncate filename (preserve extension)
const truncateFilename = (filename, maxLength) => {
  if (!filename) return 'Unknown File'
  if (filename.length <= maxLength) return filename

  const ext = filename.includes('.') ? '.' + filename.split('.').pop() : ''
  const nameWithoutExt = filename.slice(0, filename.length - ext.length)
  const truncatedName = nameWithoutExt.slice(0, maxLength - ext.length - 3) + '...'
  return truncatedName + ext
}

// Open project details modal
const navigateToProject = (simulation) => {
  selectedProject.value = simulation
}

// Close modal
const closeModal = () => {
  selectedProject.value = null
}

// Navigate to Graph Construction page (Project)
const goToProject = () => {
  if (selectedProject.value?.project_id) {
    router.push({
      name: 'Process',
      params: { projectId: selectedProject.value.project_id }
    })
    closeModal()
  }
}

// Navigate to Environment Setup page (Simulation)
const goToSimulation = () => {
  if (selectedProject.value?.simulation_id) {
    router.push({
      name: 'Simulation',
      params: { simulationId: selectedProject.value.simulation_id }
    })
    closeModal()
  }
}

// Navigate to Analysis Report page (Report)
const goToReport = () => {
  if (selectedProject.value?.report_id) {
    router.push({
      name: 'Report',
      params: { reportId: selectedProject.value.report_id }
    })
    closeModal()
  }
}

// Load history projects
const loadHistory = async () => {
  try {
    loading.value = true
    const response = await getSimulationHistory(20)
    if (response.success) {
      projects.value = response.data || []
    }
  } catch (error) {
    console.error('Failed to load history projects:', error)
    projects.value = []
  } finally {
    loading.value = false
  }
}

// Initialize IntersectionObserver
const initObserver = () => {
  if (observer) {
    observer.disconnect()
  }

  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        const shouldExpand = entry.isIntersecting

        // Update target state to execute (always record latest target state regardless of animation)
        pendingState = shouldExpand

        // Clear previous debounce timer (new scroll intent overrides old)
        if (expandDebounceTimer) {
          clearTimeout(expandDebounceTimer)
          expandDebounceTimer = null
        }

        // If animating, only record state and process after animation completes
        if (isAnimating) return

        // If target state matches current state, no need to process
        if (shouldExpand === isExpanded.value) {
          pendingState = null
          return
        }

        // Use debounce to delay state switch and prevent rapid flickering
        // Shorter delay when expanding (50ms), longer when collapsing (200ms) for stability
        const delay = shouldExpand ? 50 : 200

        expandDebounceTimer = setTimeout(() => {
          // Check if animating
          if (isAnimating) return

          // Check if pending state still needs execution (may have been overridden by subsequent scrolls)
          if (pendingState === null || pendingState === isExpanded.value) return

          // Set animation lock
          isAnimating = true
          isExpanded.value = pendingState
          pendingState = null

          // Release lock after animation completes and check for pending state changes
          setTimeout(() => {
            isAnimating = false

            // After animation, check for new pending state
            if (pendingState !== null && pendingState !== isExpanded.value) {
              // Delay slightly before executing to avoid too-quick switching
              expandDebounceTimer = setTimeout(() => {
                if (pendingState !== null && pendingState !== isExpanded.value) {
                  isAnimating = true
                  isExpanded.value = pendingState
                  pendingState = null
                  setTimeout(() => {
                    isAnimating = false
                  }, 750)
                }
              }, 100)
            }
          }, 750)
        }, delay)
      })
    },
    {
      // Use multiple thresholds for smoother detection
      threshold: [0.4, 0.6, 0.8],
      // Adjust rootMargin: shrink viewport bottom upward, require more scrolling to trigger expansion
      rootMargin: '0px 0px -150px 0px'
    }
  )

  // Start observing
  if (historyContainer.value) {
    observer.observe(historyContainer.value)
  }
}

// Watch for route changes and reload data when returning to home page
watch(() => route.path, (newPath) => {
  if (newPath === '/') {
    loadHistory()
  }
})

onMounted(async () => {
  // Ensure DOM rendering is complete before loading data
  await nextTick()
  await loadHistory()

  // Initialize observer after DOM rendering
  setTimeout(() => {
    initObserver()
  }, 100)
})

// If using keep-alive, reload data when component is activated
onActivated(() => {
  loadHistory()
})

onUnmounted(() => {
  // Clean up Intersection Observer
  if (observer) {
    observer.disconnect()
    observer = null
  }
  // Clean up debounce timer
  if (expandDebounceTimer) {
    clearTimeout(expandDebounceTimer)
    expandDebounceTimer = null
  }
})
</script>

<style scoped>
/* Container */
.history-database {
  position: relative;
  width: 100%;
  min-height: 280px;
  margin-top: 40px;
  padding: 35px 0 40px;
  overflow: visible;
}

/* Simplified display when no projects */
.history-database.no-projects {
  min-height: auto;
  padding: 40px 0 20px;
}

/* Technical grid background */
.tech-grid-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  pointer-events: none;
}

/* Use CSS background pattern to create fixed-spacing square grid */
.grid-pattern {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image:
    linear-gradient(to right, rgba(148, 163, 184, 0.08) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(148, 163, 184, 0.08) 1px, transparent 1px);
  background-size: 50px 50px;
  background-position: top left;
}

.gradient-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    linear-gradient(to right, rgba(255, 255, 255, 0.9) 0%, transparent 15%, transparent 85%, rgba(255, 255, 255, 0.9) 100%),
    linear-gradient(to bottom, rgba(255, 255, 255, 0.8) 0%, transparent 20%, transparent 80%, rgba(255, 255, 255, 0.8) 100%);
  pointer-events: none;
}

/* Title section */
.section-header {
  position: relative;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
  margin-bottom: 24px;
  font-family: var(--font-mono);
  padding: 0 40px;
}

.section-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--border), transparent);
  max-width: 300px;
}

.section-title {
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--text-muted);
  letter-spacing: 3px;
  text-transform: uppercase;
}

/* Card container */
.cards-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 0 40px;
  transition: min-height 700ms cubic-bezier(0.23, 1, 0.32, 1);
}

/* Project card */
.project-card {
  position: absolute;
  width: 280px;
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: 0;
  padding: 14px;
  cursor: pointer;
  box-shadow: var(--shadow-sm);
  transition: box-shadow 0.3s ease, border-color 0.3s ease, transform 700ms cubic-bezier(0.23, 1, 0.32, 1), opacity 700ms cubic-bezier(0.23, 1, 0.32, 1);
}

.project-card:hover {
  box-shadow: var(--shadow-lg);
  border-color: var(--border-accent);
  z-index: 1000 !important;
}

.project-card.hovering {
  z-index: 1000 !important;
}

/* Card header */
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
  font-family: var(--font-mono);
  font-size: 0.7rem;
}

.card-id {
  color: var(--text-muted);
  letter-spacing: 0.5px;
  font-weight: 500;
}

/* Feature status icon group */
.card-status-icons {
  display: flex;
  align-items: center;
  gap: 6px;
}

.status-icon {
  font-size: 0.75rem;
  transition: all 0.2s ease;
  cursor: default;
}

.status-icon.available {
  opacity: 1;
}

/* Different colors for different features */
.status-icon:nth-child(1).available { color: #60A5FA; } /* Graph Construction - Blue */
.status-icon:nth-child(2).available { color: #FBBF24; } /* Environment Setup - Amber */
.status-icon:nth-child(3).available { color: #34D399; } /* Analysis Report - Emerald */

.status-icon.unavailable {
  color: var(--text-muted);
  opacity: 0.3;
}

/* Round progress display */
.card-progress {
  display: flex;
  align-items: center;
  gap: 6px;
  letter-spacing: 0.5px;
  font-weight: 600;
  font-size: 0.65rem;
}

.status-dot {
  font-size: 0.5rem;
}

/* Progress status colors */
.card-progress.completed { color: var(--emerald); }
.card-progress.in-progress { color: var(--amber); }
.card-progress.not-started { color: var(--text-muted); }
.card-status.pending { color: var(--text-muted); }

/* File list area */
.card-files-wrapper {
  position: relative;
  width: 100%;
  min-height: 48px;
  max-height: 110px;
  margin-bottom: 12px;
  padding: 8px 10px;
  background: var(--bg-elevated);
  border-radius: 4px;
  border: 1px solid var(--border);
  overflow: hidden;
}

.files-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* More files indicator */
.files-more {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 3px 6px;
  font-family: var(--font-mono);
  font-size: 0.6rem;
  color: var(--text-muted);
  background: var(--bg-surface);
  border-radius: 3px;
  letter-spacing: 0.3px;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 6px;
  background: var(--bg-surface);
  border-radius: 3px;
  transition: all 0.2s ease;
}

.file-item:hover {
  background: var(--bg-card-hover);
  transform: translateX(2px);
}

/* Minimalist file tag style */
.file-tag {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  height: 16px;
  padding: 0 4px;
  border-radius: 2px;
  font-family: var(--font-mono);
  font-size: 0.55rem;
  font-weight: 600;
  line-height: 1;
  text-transform: uppercase;
  letter-spacing: 0.2px;
  flex-shrink: 0;
  min-width: 28px;
}

/* Dark mode file tag colors */
.file-tag.pdf { background: rgba(239, 68, 68, 0.15); color: #F87171; }
.file-tag.doc { background: rgba(59, 130, 246, 0.15); color: #60A5FA; }
.file-tag.xls { background: rgba(34, 197, 94, 0.15); color: #4ADE80; }
.file-tag.ppt { background: rgba(249, 115, 22, 0.15); color: #FB923C; }
.file-tag.txt { background: var(--bg-elevated); color: var(--text-muted); }
.file-tag.code { background: rgba(139, 92, 246, 0.15); color: #A78BFA; }
.file-tag.img { background: rgba(6, 182, 212, 0.15); color: #22D3EE; }
.file-tag.zip { background: rgba(234, 179, 8, 0.15); color: #FACC15; }
.file-tag.other { background: var(--bg-elevated); color: var(--text-muted); }

.file-name {
  font-family: var(--font-sans);
  font-size: 0.7rem;
  color: var(--text-secondary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: 0.1px;
}

/* Placeholder when no files */
.files-empty {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  height: 48px;
  color: var(--text-muted);
}

.empty-file-icon {
  font-size: 1rem;
  opacity: 0.5;
}

.empty-file-text {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  letter-spacing: 0.5px;
}

/* Hover effect for file area */
.project-card:hover .card-files-wrapper {
  border-color: var(--border-accent);
  background: var(--bg-card);
}

/* Corner decoration */
.corner-mark.top-left-only {
  position: absolute;
  top: 6px;
  left: 6px;
  width: 8px;
  height: 8px;
  border-top: 1.5px solid var(--accent);
  border-left: 1.5px solid var(--accent);
  pointer-events: none;
  z-index: 10;
}

/* Card title */
.card-title {
  font-family: var(--font-sans);
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0 0 6px 0;
  line-height: 1.4;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: color 0.3s ease;
}

.project-card:hover .card-title {
  color: var(--accent);
}

/* Card description */
.card-desc {
  font-family: var(--font-sans);
  font-size: 0.75rem;
  color: var(--text-muted);
  margin: 0 0 16px 0;
  line-height: 1.5;
  height: 34px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

/* Card footer */
.card-footer {
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 12px;
  border-top: 1px solid var(--border);
  font-family: var(--font-mono);
  font-size: 0.65rem;
  color: var(--text-muted);
  font-weight: 500;
}

/* Date-time combination */
.card-datetime {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Bottom round progress display */
.card-footer .card-progress {
  display: flex;
  align-items: center;
  gap: 6px;
  letter-spacing: 0.5px;
  font-weight: 600;
  font-size: 0.65rem;
}

.card-footer .status-dot {
  font-size: 0.5rem;
}

/* Progress status colors - bottom */
.card-footer .card-progress.completed { color: var(--emerald); }
.card-footer .card-progress.in-progress { color: var(--amber); }
.card-footer .card-progress.not-started { color: var(--text-muted); }

/* Bottom decoration line */
.card-bottom-line {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  width: 0;
  background: linear-gradient(90deg, var(--accent), var(--accent));
  transition: width 0.5s cubic-bezier(0.23, 1, 0.32, 1);
  z-index: 20;
}

.project-card:hover .card-bottom-line {
  width: 100%;
}

/* Empty state */
.empty-state, .loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  padding: 48px;
  color: var(--text-muted);
}

.empty-icon {
  font-size: 2rem;
  opacity: 0.5;
}

.loading-spinner {
  width: 24px;
  height: 24px;
  border: 2px solid var(--bg-elevated);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Responsive */
@media (max-width: 1200px) {
  .project-card {
    width: 240px;
  }
}

@media (max-width: 768px) {
  .cards-container {
    padding: 0 20px;
  }
  .project-card {
    width: 200px;
  }
}

/* ===== Simulation Playback Details Modal Styles ===== */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.modal-content {
  background: var(--bg-surface);
  width: 560px;
  max-width: 90vw;
  max-height: 85vh;
  overflow-y: auto;
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-lg);
}

/* Animation transition */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-content {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.modal-leave-active .modal-content {
  transition: all 0.2s ease-in;
}

.modal-enter-from .modal-content {
  transform: scale(0.95) translateY(10px);
  opacity: 0;
}

.modal-leave-to .modal-content {
  transform: scale(0.95) translateY(10px);
  opacity: 0;
}

/* Modal header */
.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 32px;
  border-bottom: 1px solid var(--border);
  background: var(--bg-surface);
}

.modal-title-section {
  display: flex;
  align-items: center;
  gap: 16px;
}

.modal-id {
  font-family: var(--font-mono);
  font-size: 1rem;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: 0.5px;
}

.modal-progress {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: var(--font-mono);
  font-size: 0.75rem;
  font-weight: 600;
  padding: 4px 8px;
  border-radius: 4px;
  background: var(--bg-elevated);
}

.modal-progress.completed { color: var(--emerald); background: rgba(16, 185, 129, 0.1); }
.modal-progress.in-progress { color: var(--amber); background: rgba(245, 158, 11, 0.1); }
.modal-progress.not-started { color: var(--text-muted); background: var(--bg-elevated); }

.modal-create-time {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  color: var(--text-muted);
  letter-spacing: 0.3px;
}

.modal-close {
  width: 32px;
  height: 32px;
  border: none;
  background: transparent;
  font-size: 1.5rem;
  color: var(--text-muted);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  border-radius: 6px;
}

.modal-close:hover {
  background: var(--bg-elevated);
  color: var(--text-primary);
}

/* Modal content */
.modal-body {
  padding: 24px 32px;
}

.modal-section {
  margin-bottom: 24px;
}

.modal-section:last-child {
  margin-bottom: 0;
}

.modal-label {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 10px;
  font-weight: 500;
}

.modal-requirement {
  font-size: 0.95rem;
  color: var(--text-secondary);
  line-height: 1.6;
  padding: 16px;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: 8px;
}

.modal-files {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 200px;
  overflow-y: auto;
  padding-right: 4px;
}

/* Custom scrollbar style */
.modal-files::-webkit-scrollbar {
  width: 4px;
}

.modal-files::-webkit-scrollbar-track {
  background: var(--bg-elevated);
  border-radius: 2px;
}

.modal-files::-webkit-scrollbar-thumb {
  background: rgba(148, 163, 184, 0.3);
  border-radius: 2px;
}

.modal-files::-webkit-scrollbar-thumb:hover {
  background: rgba(148, 163, 184, 0.5);
}

.modal-file-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 14px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 6px;
  transition: all 0.2s ease;
}

.modal-file-item:hover {
  border-color: var(--border-accent);
  box-shadow: var(--shadow-sm);
}

.modal-file-name {
  font-size: 0.85rem;
  color: var(--text-secondary);
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.modal-empty {
  font-size: 0.85rem;
  color: var(--text-muted);
  padding: 16px;
  background: var(--bg-elevated);
  border: 1px dashed var(--border);
  border-radius: 6px;
  text-align: center;
}

/* Simulation playback divider */
.modal-divider {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 10px 32px 0;
  background: var(--bg-surface);
}

.divider-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--border), transparent);
}

.divider-text {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  color: var(--text-muted);
  letter-spacing: 2px;
  text-transform: uppercase;
  white-space: nowrap;
}

/* Navigation buttons */
.modal-actions {
  display: flex;
  gap: 16px;
  padding: 20px 32px;
  background: var(--bg-surface);
}

.modal-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 16px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--bg-surface);
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
  overflow: hidden;
}

.modal-btn:hover:not(:disabled) {
  border-color: var(--accent);
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.modal-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  background: var(--bg-elevated);
}

.btn-step {
  font-family: var(--font-mono);
  font-size: 0.6rem;
  font-weight: 500;
  color: var(--text-muted);
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.btn-icon {
  font-size: 1.4rem;
  line-height: 1;
  transition: color 0.2s ease;
}

.btn-text {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  color: var(--text-secondary);
}

.modal-btn.btn-project .btn-icon { color: #60A5FA; }
.modal-btn.btn-simulation .btn-icon { color: #FBBF24; }
.modal-btn.btn-report .btn-icon { color: #34D399; }

.modal-btn:hover:not(:disabled) .btn-text {
  color: var(--text-primary);
}

/* Playback unavailable notice */
.modal-playback-hint {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 32px 20px;
  background: var(--bg-surface);
}

.hint-text {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  color: var(--text-muted);
  letter-spacing: 0.3px;
  text-align: center;
  line-height: 1.5;
}
</style>
