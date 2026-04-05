<template>
  <div class="process-page">
    <!-- Top navigation bar -->
    <nav class="navbar">
      <div class="nav-brand" @click="goHome">REGUSIM</div>
      
      <!-- Center step indicator -->
      <div class="nav-center">
        <div class="step-badge">STEP 01</div>
        <div class="step-name">Graph Build</div>
      </div>

      <div class="nav-status">
        <span class="status-dot" :class="statusClass"></span>
        <span class="status-text">{{ statusText }}</span>
      </div>
    </nav>

    <!-- Main content area -->
    <div class="main-content">
      <!-- Left side: Real-time graph display -->
      <div class="left-panel" :class="{ 'full-screen': isFullScreen }">
        <div class="panel-header">
          <div class="header-left">
            <span class="header-deco">◆</span>
            <span class="header-title">Live Knowledge Graph</span>
          </div>
          <div class="header-right">
            <template v-if="graphData">
              <span class="stat-item">{{ graphData.node_count || graphData.nodes?.length || 0 }} Nodes</span>
              <span class="stat-divider">|</span>
              <span class="stat-item">{{ graphData.edge_count || graphData.edges?.length || 0 }} Edges</span>
              <span class="stat-divider">|</span>
            </template>
            <div class="action-buttons">
                <button class="action-btn" @click="refreshGraph" :disabled="graphLoading" title="Refresh Graph">
                  <span class="icon-refresh" :class="{ 'spinning': graphLoading }">↻</span>
                </button>
                <button class="action-btn" @click="toggleFullScreen" :title="isFullScreen ? 'Exit Fullscreen' : 'Fullscreen'">
                  <span class="icon-fullscreen">{{ isFullScreen ? '↙' : '↗' }}</span>
                </button>
            </div>
          </div>
        </div>
        
        <div class="graph-container" ref="graphContainer">
          <!-- Graph visualization (show as long as there is data) -->
          <div v-if="graphData" class="graph-view">
            <svg ref="graphSvg" class="graph-svg"></svg>
            <!-- Building in progress hint -->
            <div v-if="currentPhase === 1" class="graph-building-hint">
              <span class="building-dot"></span>
              Real-time update in progress...
            </div>
            
            <!-- Node/Edge detail panel -->
            <div v-if="selectedItem" class="detail-panel">
              <div class="detail-panel-header">
                <span class="detail-title">{{ selectedItem.type === 'node' ? 'Node Details' : 'Relationship' }}</span>
                <span v-if="selectedItem.type === 'node'" class="detail-badge" :style="{ background: selectedItem.color }">
                  {{ selectedItem.entityType }}
                </span>
                <button class="detail-close" @click="closeDetailPanel">×</button>
              </div>
              
              <!-- Node details -->
              <div v-if="selectedItem.type === 'node'" class="detail-content">
                <div class="detail-row">
                  <span class="detail-label">Name:</span>
                  <span class="detail-value highlight">{{ selectedItem.data.name }}</span>
                </div>
                <div class="detail-row">
                  <span class="detail-label">UUID:</span>
                  <span class="detail-value uuid">{{ selectedItem.data.uuid }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.created_at">
                  <span class="detail-label">Created:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.created_at) }}</span>
                </div>
                
                <!-- Properties / Attributes -->
                <div class="detail-section" v-if="selectedItem.data.attributes && Object.keys(selectedItem.data.attributes).length > 0">
                  <span class="detail-label">Properties:</span>
                  <div class="properties-list">
                    <div v-for="(value, key) in selectedItem.data.attributes" :key="key" class="property-item">
                      <span class="property-key">{{ key }}:</span>
                      <span class="property-value">{{ value }}</span>
                    </div>
                  </div>
                </div>
                
                <!-- Summary -->
                <div class="detail-section" v-if="selectedItem.data.summary">
                  <span class="detail-label">Summary:</span>
                  <p class="detail-summary">{{ selectedItem.data.summary }}</p>
                </div>
                
                <!-- Labels -->
                <div class="detail-row" v-if="selectedItem.data.labels?.length">
                  <span class="detail-label">Labels:</span>
                  <div class="detail-labels">
                    <span v-for="label in selectedItem.data.labels" :key="label" class="label-tag">{{ label }}</span>
                  </div>
                </div>
              </div>
              
              <!-- Edge details -->
              <div v-else class="detail-content">
                <!-- Relationship display -->
                <div class="edge-relation">
                  <span class="edge-source">{{ selectedItem.data.source_name || selectedItem.data.source_node_name }}</span>
                  <span class="edge-arrow">→</span>
                  <span class="edge-type">{{ selectedItem.data.name || selectedItem.data.fact_type || 'RELATED_TO' }}</span>
                  <span class="edge-arrow">→</span>
                  <span class="edge-target">{{ selectedItem.data.target_name || selectedItem.data.target_node_name }}</span>
                </div>
                
                <div class="detail-subtitle">Relationship</div>
                
                <div class="detail-row">
                  <span class="detail-label">UUID:</span>
                  <span class="detail-value uuid">{{ selectedItem.data.uuid }}</span>
                </div>
                <div class="detail-row">
                  <span class="detail-label">Label:</span>
                  <span class="detail-value">{{ selectedItem.data.name || selectedItem.data.fact_type || 'RELATED_TO' }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.fact_type">
                  <span class="detail-label">Type:</span>
                  <span class="detail-value">{{ selectedItem.data.fact_type }}</span>
                </div>
                
                <!-- Fact -->
                <div class="detail-section" v-if="selectedItem.data.fact">
                  <span class="detail-label">Fact:</span>
                  <p class="detail-summary">{{ selectedItem.data.fact }}</p>
                </div>
                
                <!-- Episodes -->
                <div class="detail-section" v-if="selectedItem.data.episodes?.length">
                  <span class="detail-label">Episodes:</span>
                  <div class="episodes-list">
                    <span v-for="ep in selectedItem.data.episodes" :key="ep" class="episode-tag">{{ ep }}</span>
                  </div>
                </div>
                
                <div class="detail-row" v-if="selectedItem.data.created_at">
                  <span class="detail-label">Created:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.created_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.valid_at">
                  <span class="detail-label">Valid From:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.valid_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.invalid_at">
                  <span class="detail-label">Invalid At:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.invalid_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.expired_at">
                  <span class="detail-label">Expired At:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.expired_at) }}</span>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Loading state -->
          <div v-else-if="graphLoading" class="graph-loading">
            <div class="loading-animation">
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
            </div>
            <p class="loading-text">Loading graph data...</p>
          </div>
          
          <!-- Waiting for build -->
          <div v-else-if="currentPhase < 1" class="graph-waiting">
            <div class="waiting-icon">
              <svg viewBox="0 0 100 100" class="network-icon">
                <circle cx="50" cy="20" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="20" cy="60" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="80" cy="60" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="50" cy="80" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <line x1="50" y1="28" x2="25" y2="54" stroke="#000" stroke-width="1"/>
                <line x1="50" y1="28" x2="75" y2="54" stroke="#000" stroke-width="1"/>
                <line x1="28" y1="60" x2="72" y2="60" stroke="#000" stroke-width="1" stroke-dasharray="4"/>
                <line x1="50" y1="72" x2="26" y2="66" stroke="#000" stroke-width="1"/>
                <line x1="50" y1="72" x2="74" y2="66" stroke="#000" stroke-width="1"/>
              </svg>
            </div>
            <p class="waiting-text">Waiting for ontology generation</p>
            <p class="waiting-hint">Graph construction will start automatically after generation completes</p>
          </div>
          
          <!-- Building in progress without data -->
          <div v-else-if="currentPhase === 1 && !graphData" class="graph-waiting">
            <div class="loading-animation">
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
            </div>
            <p class="waiting-text">Building graph</p>
            <p class="waiting-hint">Data will be displayed soon...</p>
          </div>
          
          <!-- Error state -->
          <div v-else-if="error" class="graph-error">
            <span class="error-icon">⚠</span>
            <p>{{ error }}</p>
          </div>
        </div>
        
        <!-- Graph legend -->
        <div v-if="graphData" class="graph-legend">
          <div class="legend-item" v-for="type in entityTypes" :key="type.name">
            <span class="legend-dot" :style="{ background: type.color }"></span>
            <span class="legend-label">{{ type.name }}</span>
            <span class="legend-count">{{ type.count }}</span>
          </div>
        </div>
      </div>

      <!-- Right side: Build process details -->
      <div class="right-panel" :class="{ 'hidden': isFullScreen }">
        <div class="panel-header dark-header">
          <span class="header-icon">▣</span>
          <span class="header-title">Build Process</span>
        </div>

        <div class="process-content">
          <!-- Phase 1: Ontology Generation -->
          <div class="process-phase" :class="{ 'active': currentPhase === 0, 'completed': currentPhase > 0 }">
            <div class="phase-header">
              <span class="phase-num">01</span>
              <div class="phase-info">
                <div class="phase-title">Ontology Generation</div>
                <div class="phase-api">/api/graph/ontology/generate</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(0)">
                {{ getPhaseStatusText(0) }}
              </span>
            </div>
            
            <div class="phase-detail">
              <div class="detail-section">
                <div class="detail-label">Description</div>
                <div class="detail-content">
                  After uploading documents, LLM analyzes the content and automatically generates an ontology structure suitable for knowledge graph simulation (entity types + relationship types)
                </div>
              </div>
              
              <!-- Ontology generation progress -->
              <div class="detail-section" v-if="ontologyProgress && currentPhase === 0">
                <div class="detail-label">Generation Progress</div>
                <div class="ontology-progress">
                  <div class="progress-spinner"></div>
                  <span class="progress-text">{{ ontologyProgress.message }}</span>
                </div>
              </div>
              
              <!-- Generated ontology information -->
              <div class="detail-section" v-if="projectData?.ontology">
                <div class="detail-label">Generated Entity Types ({{ projectData.ontology.entity_types?.length || 0 }})</div>
                <div class="entity-tags">
                  <span 
                    v-for="entity in projectData.ontology.entity_types" 
                    :key="entity.name"
                    class="entity-tag"
                  >
                    {{ entity.name }}
                  </span>
                </div>
              </div>
              
              <div class="detail-section" v-if="projectData?.ontology">
                <div class="detail-label">Generated Relationship Types ({{ projectData.ontology.relation_types?.length || 0 }})</div>
                <div class="relation-list">
                  <div 
                    v-for="(rel, idx) in projectData.ontology.relation_types?.slice(0, 5) || []" 
                    :key="idx"
                    class="relation-item"
                  >
                    <span class="rel-source">{{ rel.source_type }}</span>
                    <span class="rel-arrow">→</span>
                    <span class="rel-name">{{ rel.name }}</span>
                    <span class="rel-arrow">→</span>
                    <span class="rel-target">{{ rel.target_type }}</span>
                  </div>
                  <div v-if="(projectData.ontology.relation_types?.length || 0) > 5" class="relation-more">
                    +{{ projectData.ontology.relation_types.length - 5 }} more relationships...
                  </div>
                </div>
              </div>
              
              <!-- Waiting state -->
              <div class="detail-section waiting-state" v-if="!projectData?.ontology && currentPhase === 0 && !ontologyProgress">
                <div class="waiting-hint">Waiting for ontology generation...</div>
              </div>
            </div>
          </div>

          <!-- Phase 2: Graph Build -->
          <div class="process-phase" :class="{ 'active': currentPhase === 1, 'completed': currentPhase > 1 }">
            <div class="phase-header">
              <span class="phase-num">02</span>
              <div class="phase-info">
                <div class="phase-title">Graph Build</div>
                <div class="phase-api">/api/graph/build</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(1)">
                {{ getPhaseStatusText(1) }}
              </span>
            </div>
            
            <div class="phase-detail">
              <div class="detail-section">
                <div class="detail-label">Description</div>
                <div class="detail-content">
                  Based on the generated ontology, the documents are chunked and the Neo4j API is called to build the knowledge graph, extracting entities and relationships
                </div>
              </div>
              
              <!-- Waiting for ontology completion -->
              <div class="detail-section waiting-state" v-if="currentPhase < 1">
                <div class="waiting-hint">Waiting for ontology generation to complete...</div>
              </div>
              
              <!-- Build progress -->
              <div class="detail-section" v-if="buildProgress && currentPhase >= 1">
                <div class="detail-label">Build Progress</div>
                <div class="progress-bar">
                  <div class="progress-fill" :style="{ width: buildProgress.progress + '%' }"></div>
                </div>
                <div class="progress-info">
                  <span class="progress-message">{{ buildProgress.message }}</span>
                  <span class="progress-percent">{{ buildProgress.progress }}%</span>
                </div>
              </div>
              
              <div class="detail-section" v-if="graphData">
                <div class="detail-label">Build Result</div>
                <div class="build-result">
                  <div class="result-item">
                    <span class="result-value">{{ graphData.node_count }}</span>
                    <span class="result-label">Entity Nodes</span>
                  </div>
                  <div class="result-item">
                    <span class="result-value">{{ graphData.edge_count }}</span>
                    <span class="result-label">Relationship Edges</span>
                  </div>
                  <div class="result-item">
                    <span class="result-value">{{ entityTypes.length }}</span>
                    <span class="result-label">Entity Types</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Phase 3: Complete -->
          <div class="process-phase" :class="{ 'active': currentPhase === 2, 'completed': currentPhase > 2 }">
            <div class="phase-header">
              <span class="phase-num">03</span>
              <div class="phase-info">
                <div class="phase-title">Build Complete</div>
                <div class="phase-api">Ready for next step</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(2)">
                {{ getPhaseStatusText(2) }}
              </span>
            </div>
          </div>

          <!-- Next step button -->
          <div class="next-step-section" v-if="currentPhase >= 2">
            <button class="next-step-btn" @click="goToNextStep" :disabled="currentPhase < 2">
              Enter Environment Setup
              <span class="btn-arrow">→</span>
            </button>
          </div>
        </div>

        <!-- Project info panel -->
        <div class="project-panel">
          <div class="project-header">
            <span class="project-icon">◇</span>
            <span class="project-title">Project Information</span>
          </div>
          <div class="project-details" v-if="projectData">
            <div class="project-item">
              <span class="item-label">Project Name</span>
              <span class="item-value">{{ projectData.name }}</span>
            </div>
            <div class="project-item">
              <span class="item-label">Project ID</span>
              <span class="item-value code">{{ projectData.project_id }}</span>
            </div>
            <div class="project-item" v-if="projectData.graph_id">
              <span class="item-label">Graph ID</span>
              <span class="item-value code">{{ projectData.graph_id }}</span>
            </div>
            <div class="project-item">
              <span class="item-label">Simulation Requirement</span>
              <span class="item-value">{{ projectData.simulation_requirement || '-' }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { generateOntology, getProject, buildGraph, getTaskStatus, getGraphData } from '../api/graph'
import { getPendingUpload, clearPendingUpload } from '../store/pendingUpload'
import * as d3 from 'd3'

const route = useRoute()
const router = useRouter()

// Current project ID (may change from 'new' to actual ID)
const currentProjectId = ref(route.params.projectId)

// State
const loading = ref(true)
const graphLoading = ref(false)
const error = ref('')
const projectData = ref(null)
const graphData = ref(null)
const buildProgress = ref(null)
const ontologyProgress = ref(null) // Ontology generation progress
const currentPhase = ref(-1) // -1: Uploading, 0: Generating ontology, 1: Building graph, 2: Complete
const selectedItem = ref(null) // Selected node or edge
const isFullScreen = ref(false)

// DOM refs
const graphContainer = ref(null)
const graphSvg = ref(null)

// Polling timers
let pollTimer = null

// Computed properties
const statusClass = computed(() => {
  if (error.value) return 'error'
  if (currentPhase.value >= 2) return 'completed'
  return 'processing'
})

const statusText = computed(() => {
  if (error.value) return 'Build Failed'
  if (currentPhase.value >= 2) return 'Build Complete'
  if (currentPhase.value === 1) return 'Building Graph'
  if (currentPhase.value === 0) return 'Generating Ontology'
  return 'Initializing'
})

const entityTypes = computed(() => {
  if (!graphData.value?.nodes) return []
  
  const typeMap = {}
  const colors = ['#FF6B35', '#004E89', '#7B2D8E', '#1A936F', '#C5283D', '#E9724C']
  
  graphData.value.nodes.forEach(node => {
    const type = node.labels?.find(l => l !== 'Entity') || 'Entity'
    if (!typeMap[type]) {
      typeMap[type] = { name: type, count: 0, color: colors[Object.keys(typeMap).length % colors.length] }
    }
    typeMap[type].count++
  })
  
  return Object.values(typeMap)
})

// Methods
const goHome = () => {
  router.push('/')
}

const goToNextStep = () => {
  // TODO: Enter environment setup step
  alert('Environment setup in development...')
}

const toggleFullScreen = () => {
  isFullScreen.value = !isFullScreen.value
  // Wait for transition to finish then re-render graph
  setTimeout(() => {
    renderGraph()
  }, 350)
}

// Close detail panel
const closeDetailPanel = () => {
  selectedItem.value = null
}

// Format date
const formatDate = (dateStr) => {
  if (!dateStr) return '-'
  try {
    const date = new Date(dateStr)
    return date.toLocaleString('zh-CN', {
      year: 'numeric',
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  } catch {
    return dateStr
  }
}

// Select node
const selectNode = (nodeData, color) => {
  selectedItem.value = {
    type: 'node',
    data: nodeData,
    color: color,
    entityType: nodeData.labels?.find(l => l !== 'Entity' && l !== 'Node') || 'Entity'
  }
}

// Select edge
const selectEdge = (edgeData) => {
  selectedItem.value = {
    type: 'edge',
    data: edgeData
  }
}

const getPhaseStatusClass = (phase) => {
  if (currentPhase.value > phase) return 'completed'
  if (currentPhase.value === phase) return 'active'
  return 'pending'
}

const getPhaseStatusText = (phase) => {
  if (currentPhase.value > phase) return 'Completed'
  if (currentPhase.value === phase) {
    if (phase === 1 && buildProgress.value) {
      return `${buildProgress.value.progress}%`
    }
    return 'In Progress'
  }
  return 'Waiting'
}

// Initialize - handle new or existing project
const initProject = async () => {
  const paramProjectId = route.params.projectId

  if (paramProjectId === 'new') {
    // New project: get pending upload data from store
    await handleNewProject()
  } else {
    // Load existing project
    currentProjectId.value = paramProjectId
    await loadProject()
  }
}

// Handle new project - call ontology/generate API
const handleNewProject = async () => {
  const pending = getPendingUpload()

  if (!pending.isPending || pending.files.length === 0) {
    error.value = 'No files pending upload. Please go back to home and try again.'
    loading.value = false
    return
  }

  try {
    loading.value = true
    currentPhase.value = 0 // Ontology generation phase
    ontologyProgress.value = { message: 'Uploading files and analyzing documents...' }

    // Build FormData
    const formDataObj = new FormData()
    pending.files.forEach(file => {
      formDataObj.append('files', file)
    })
    formDataObj.append('simulation_requirement', pending.simulationRequirement)

    // Call ontology generation API
    const response = await generateOntology(formDataObj)

    if (response.success) {
      // Clear pending upload data
      clearPendingUpload()

      // Update project ID and data
      currentProjectId.value = response.data.project_id
      projectData.value = response.data

      // Update URL (no page refresh)
      router.replace({
        name: 'Process',
        params: { projectId: response.data.project_id }
      })

      ontologyProgress.value = null

      // Automatically start graph building
      await startBuildGraph()
    } else {
      error.value = response.error || 'Ontology generation failed'
    }
  } catch (err) {
    console.error('Handle new project error:', err)
    error.value = 'Project initialization failed: ' + (err.message || 'Unknown error')
  } finally {
    loading.value = false
  }
}

// Load existing project data
const loadProject = async () => {
  try {
    loading.value = true
    const response = await getProject(currentProjectId.value)
    
    if (response.success) {
      projectData.value = response.data
      updatePhaseByStatus(response.data.status)

      // Automatically start graph building
      if (response.data.status === 'ontology_generated' && !response.data.graph_id) {
        await startBuildGraph()
      }

      // Continue polling running build tasks
      if (response.data.status === 'graph_building' && response.data.graph_build_task_id) {
        currentPhase.value = 1
        startPollingTask(response.data.graph_build_task_id)
      }

      // Load completed graphs
      if (response.data.status === 'graph_completed' && response.data.graph_id) {
        currentPhase.value = 2
        await loadGraph(response.data.graph_id)
      }
    } else {
      error.value = response.error || 'Failed to load project'
    }
  } catch (err) {
    console.error('Load project error:', err)
    error.value = 'Failed to load project: ' + (err.message || 'Unknown error')
  } finally {
    loading.value = false
  }
}

const updatePhaseByStatus = (status) => {
  switch (status) {
    case 'created':
    case 'ontology_generated':
      currentPhase.value = 0
      break
    case 'graph_building':
      currentPhase.value = 1
      break
    case 'graph_completed':
      currentPhase.value = 2
      break
    case 'failed':
      error.value = projectData.value?.error || 'Processing failed'
      break
  }
}

// Start building graph
const startBuildGraph = async () => {
  try {
    currentPhase.value = 1
    // Initialize progress
    buildProgress.value = {
      progress: 0,
      message: 'Starting graph build...'
    }

    const response = await buildGraph({ project_id: currentProjectId.value })

    if (response.success) {
      buildProgress.value.message = 'Graph build task started...'

      // Save task_id for polling
      const taskId = response.data.task_id

      // Start graph data polling (independent from task status polling)
      startGraphPolling()

      // Start task status polling
      startPollingTask(taskId)
    } else {
      error.value = response.error || 'Failed to start graph build'
      buildProgress.value = null
    }
  } catch (err) {
    console.error('Build graph error:', err)
    error.value = 'Failed to start graph build: ' + (err.message || 'Unknown error')
    buildProgress.value = null
  }
}

// Graph data polling timer
let graphPollTimer = null

// Start graph data polling
const startGraphPolling = () => {
  // Fetch once immediately
  fetchGraphData()

  // Auto-fetch graph data every 10 seconds
  graphPollTimer = setInterval(async () => {
    await fetchGraphData()
  }, 10000)
}

// Manually refresh graph
const refreshGraph = async () => {
  graphLoading.value = true
  await fetchGraphData()
  graphLoading.value = false
}

// Stop graph data polling
const stopGraphPolling = () => {
  if (graphPollTimer) {
    clearInterval(graphPollTimer)
    graphPollTimer = null
  }
}

// Fetch graph data
const fetchGraphData = async () => {
  try {
    // Get project info first to get graph_id
    const projectResponse = await getProject(currentProjectId.value)

    if (projectResponse.success && projectResponse.data.graph_id) {
      const graphId = projectResponse.data.graph_id
      projectData.value = projectResponse.data

      // Fetch graph data
      const graphResponse = await getGraphData(graphId)

      if (graphResponse.success && graphResponse.data) {
        const newData = graphResponse.data
        const newNodeCount = newData.node_count || newData.nodes?.length || 0
        const oldNodeCount = graphData.value?.node_count || graphData.value?.nodes?.length || 0

        console.log('Fetching graph data, nodes:', newNodeCount, 'edges:', newData.edge_count || newData.edges?.length || 0)

        // Update and re-render when data changes
        if (newNodeCount !== oldNodeCount || !graphData.value) {
          graphData.value = newData
          await nextTick()
          renderGraph()
        }
      }
    }
  } catch (err) {
    console.log('Graph data fetch:', err.message || 'not ready')
  }
}

// Poll task status
const startPollingTask = (taskId) => {
  // Execute query once immediately
  pollTaskStatus(taskId)

  // Then poll at intervals
  pollTimer = setInterval(() => {
    pollTaskStatus(taskId)
  }, 2000)
}

// Query task status
const pollTaskStatus = async (taskId) => {
  try {
    const response = await getTaskStatus(taskId)

    if (response.success) {
      const task = response.data

      // Update progress display
      buildProgress.value = {
        progress: task.progress || 0,
        message: task.message || 'Processing...'
      }

      console.log('Task status:', task.status, 'Progress:', task.progress)

      if (task.status === 'completed') {
        console.log('✅ Graph build complete, loading full data...')
        
        stopPolling()
        stopGraphPolling()
        currentPhase.value = 2

        // Update progress display to complete status
        buildProgress.value = {
          progress: 100,
          message: 'Build complete, loading graph...'
        }

        // Reload project data to get graph_id
        const projectResponse = await getProject(currentProjectId.value)
        if (projectResponse.success) {
          projectData.value = projectResponse.data

          // Finally load complete graph data
          if (projectResponse.data.graph_id) {
            console.log('📊 Loading complete graph:', projectResponse.data.graph_id)
            await loadGraph(projectResponse.data.graph_id)
            console.log('✅ Graph load complete')
          }
        }

        // Clear progress display
        buildProgress.value = null
      } else if (task.status === 'failed') {
        stopPolling()
        stopGraphPolling()
        error.value = 'Graph build failed: ' + (task.error || 'Unknown error')
        buildProgress.value = null
      }
    }
  } catch (err) {
    console.error('Poll task error:', err)
  }
}

const stopPolling = () => {
  if (pollTimer) {
    clearInterval(pollTimer)
    pollTimer = null
  }
}

// Load graph data
const loadGraph = async (graphId) => {
  try {
    graphLoading.value = true
    const response = await getGraphData(graphId)

    if (response.success) {
      graphData.value = response.data
      await nextTick()
      renderGraph()
    }
  } catch (err) {
    console.error('Load graph error:', err)
  } finally {
    graphLoading.value = false
  }
}

// Render graph (D3.js)
const renderGraph = () => {
  if (!graphSvg.value || !graphData.value) {
    console.log('Cannot render: svg or data missing')
    return
  }

  const container = graphContainer.value
  if (!container) {
    console.log('Cannot render: container missing')
    return
  }

  // Get container dimensions
  const rect = container.getBoundingClientRect()
  const width = rect.width || 800
  const height = (rect.height || 600) - 60

  if (width <= 0 || height <= 0) {
    console.log('Cannot render: invalid dimensions', width, height)
    return
  }

  console.log('Rendering graph:', width, 'x', height)
  
  const svg = d3.select(graphSvg.value)
    .attr('width', width)
    .attr('height', height)
    .attr('viewBox', `0 0 ${width} ${height}`)
  
  svg.selectAll('*').remove()

  // Process node data
  const nodesData = graphData.value.nodes || []
  const edgesData = graphData.value.edges || []

  if (nodesData.length === 0) {
    console.log('No nodes to render')
    // Show empty state
    svg.append('text')
      .attr('x', width / 2)
      .attr('y', height / 2)
      .attr('text-anchor', 'middle')
      .attr('fill', '#999')
      .text('Waiting for graph data...')
    return
  }

  // Create node map for name lookup
  const nodeMap = {}
  nodesData.forEach(n => {
    nodeMap[n.uuid] = n
  })

  const nodes = nodesData.map(n => ({
    id: n.uuid,
    name: n.name || 'Unnamed',
    type: n.labels?.find(l => l !== 'Entity' && l !== 'Node') || 'Entity',
    rawData: n // Save original data
  }))

  // Create node ID set for filtering valid edges
  const nodeIds = new Set(nodes.map(n => n.id))

  const edges = edgesData
    .filter(e => nodeIds.has(e.source_node_uuid) && nodeIds.has(e.target_node_uuid))
    .map(e => ({
      source: e.source_node_uuid,
      target: e.target_node_uuid,
      type: e.fact_type || e.name || 'RELATED_TO',
      rawData: {
        ...e,
        source_name: nodeMap[e.source_node_uuid]?.name || 'Unknown',
        target_name: nodeMap[e.target_node_uuid]?.name || 'Unknown'
      }
    }))

  console.log('Nodes:', nodes.length, 'Edges:', edges.length)

  // Color mapping
  const types = [...new Set(nodes.map(n => n.type))]
  const colorScale = d3.scaleOrdinal()
    .domain(types)
    .range(['#FF6B35', '#004E89', '#7B2D8E', '#1A936F', '#C5283D', '#E9724C', '#2D3436', '#6C5CE7'])

  // Force-directed layout
  const simulation = d3.forceSimulation(nodes)
    .force('link', d3.forceLink(edges).id(d => d.id).distance(100).strength(0.5))
    .force('charge', d3.forceManyBody().strength(-300))
    .force('center', d3.forceCenter(width / 2, height / 2))
    .force('collision', d3.forceCollide().radius(40))
    .force('x', d3.forceX(width / 2).strength(0.05))
    .force('y', d3.forceY(height / 2).strength(0.05))

  // Add zoom functionality
  const g = svg.append('g')

  svg.call(d3.zoom()
    .extent([[0, 0], [width, height]])
    .scaleExtent([0.2, 4])
    .on('zoom', (event) => {
      g.attr('transform', event.transform)
    }))

  // Draw edges (including clickable transparent wide lines)
  const linkGroup = g.append('g')
    .attr('class', 'links')
    .selectAll('g')
    .data(edges)
    .enter()
    .append('g')
    .style('cursor', 'pointer')
    .on('click', (event, d) => {
      event.stopPropagation()
      selectEdge(d.rawData)
    })

  // Visible thin lines
  const link = linkGroup.append('line')
    .attr('stroke', '#ccc')
    .attr('stroke-width', 1.5)
    .attr('stroke-opacity', 0.6)

  // Transparent wide lines for clicking
  linkGroup.append('line')
    .attr('stroke', 'transparent')
    .attr('stroke-width', 10)

  // Edge labels
  const linkLabel = g.append('g')
    .attr('class', 'link-labels')
    .selectAll('text')
    .data(edges)
    .enter()
    .append('text')
    .attr('font-size', '9px')
    .attr('fill', '#999')
    .attr('text-anchor', 'middle')
    .text(d => d.type.length > 15 ? d.type.substring(0, 12) + '...' : d.type)

  // Draw nodes
  const node = g.append('g')
    .attr('class', 'nodes')
    .selectAll('g')
    .data(nodes)
    .enter()
    .append('g')
    .style('cursor', 'pointer')
    .on('click', (event, d) => {
      event.stopPropagation()
      selectNode(d.rawData, colorScale(d.type))
    })
    .call(d3.drag()
      .on('start', dragstarted)
      .on('drag', dragged)
      .on('end', dragended))
  
  node.append('circle')
    .attr('r', 10)
    .attr('fill', d => colorScale(d.type))
    .attr('stroke', '#fff')
    .attr('stroke-width', 2)
    .attr('class', 'node-circle')
  
  node.append('text')
    .attr('dx', 14)
    .attr('dy', 4)
    .text(d => d.name?.substring(0, 12) || '')
    .attr('font-size', '11px')
    .attr('fill', '#333')
    .attr('font-family', 'JetBrains Mono, monospace')

  // Close detail panel when clicking blank space
  svg.on('click', () => {
    closeDetailPanel()
  })

  simulation.on('tick', () => {
    // Update position of all edges (including visible lines and transparent click areas)
    linkGroup.selectAll('line')
      .attr('x1', d => d.source.x)
      .attr('y1', d => d.source.y)
      .attr('x2', d => d.target.x)
      .attr('y2', d => d.target.y)

    // Update edge label position
    linkLabel
      .attr('x', d => (d.source.x + d.target.x) / 2)
      .attr('y', d => (d.source.y + d.target.y) / 2 - 5)

    node.attr('transform', d => `translate(${d.x},${d.y})`)
  })

  function dragstarted(event) {
    if (!event.active) simulation.alphaTarget(0.3).restart()
    event.subject.fx = event.subject.x
    event.subject.fy = event.subject.y
  }

  function dragged(event) {
    event.subject.fx = event.x
    event.subject.fy = event.y
  }

  function dragended(event) {
    if (!event.active) simulation.alphaTarget(0)
    event.subject.fx = null
    event.subject.fy = null
  }
}

// Watch graph data changes
watch(graphData, () => {
  if (graphData.value) {
    nextTick(() => renderGraph())
  }
})

// Lifecycle
onMounted(() => {
  initProject()
})

onUnmounted(() => {
  stopPolling()
  stopGraphPolling()
})
</script>

<style scoped>
.process-page {
  min-height: 100vh;
  background: var(--bg-base);
  overflow: hidden;
}

/* Navigation Bar */
.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  height: 52px;
  background: var(--bg-surface);
  border-bottom: 1px solid var(--border);
  color: var(--text-primary);
  z-index: 10;
  position: relative;
}

.nav-brand {
  font-size: 0.95rem;
  font-weight: 700;
  font-family: var(--font-serif);
  letter-spacing: 0.3px;
  cursor: pointer;
  transition: color 0.2s;
}

.nav-brand:hover {
  color: var(--accent);
}

.nav-center {
  display: flex;
  align-items: center;
  gap: 10px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.step-badge {
  background: var(--text-primary);
  color: var(--text-inverse);
  padding: 2px 10px;
  font-size: 0.68rem;
  font-weight: 700;
  font-family: var(--font-mono);
  letter-spacing: 0.05em;
  border-radius: var(--radius-sm);
}

.step-name {
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--text-primary);
}

.nav-status {
  display: flex;
  align-items: center;
}

.status-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--text-muted);
  margin-right: 8px;
}

.status-dot.processing {
  background: var(--accent);
  animation: pulse 1.5s infinite;
}

.status-dot.completed {
  background: var(--emerald);
}

.status-dot.error {
  background: var(--rose);
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.status-text {
  font-size: 0.72rem;
  color: var(--text-secondary);
}

/* Main Content Area */
.main-content {
  display: flex;
  height: calc(100vh - 52px);
  position: relative;
}

.left-panel {
  width: 50%;
  flex: none;
  display: flex;
  flex-direction: column;
  border-right: 1px solid var(--border);
  transition: width 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  background: var(--bg-base);
  z-index: 5;
}

.left-panel.full-screen {
  width: 100%;
  border-right: none;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 20px;
  border-bottom: 1px solid var(--border);
  background: var(--bg-surface);
  height: 46px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 8px;
}

.header-deco {
  color: var(--accent);
  font-size: 0.7rem;
}

.header-title {
  font-size: 0.82rem;
  font-weight: 600;
  font-family: var(--font-serif);
}

.header-right {
  display: flex;
  align-items: center;
  gap: 14px;
  font-size: 0.72rem;
  color: var(--text-secondary);
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

.stat-val {
  font-weight: 600;
  color: var(--text-primary);
}

.stat-divider {
  color: var(--border);
}

.action-buttons {
    display: flex;
    align-items: center;
    gap: 6px;
}

.action-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  background: transparent;
  border: 1px solid var(--border);
  cursor: pointer;
  transition: all 0.2s;
  color: var(--text-muted);
  border-radius: var(--radius-sm);
}

.action-btn:hover:not(:disabled) {
  background: var(--bg-elevated);
  color: var(--accent);
  border-color: var(--accent-border);
}

.action-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.icon-refresh, .icon-fullscreen {
  font-size: 1rem;
  line-height: 1;
}

.icon-refresh.spinning {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Graph Container */
.graph-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  background: var(--bg-warm);
}

.graph-loading,
.graph-waiting,
.graph-error {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.loading-animation {
  position: relative;
  width: 80px;
  height: 80px;
  margin: 0 auto 20px;
}

.loading-ring {
  position: absolute;
  border: 2px solid transparent;
  border-radius: 50%;
  animation: ring-rotate 1.5s linear infinite;
}

.loading-ring:nth-child(1) {
  width: 80px;
  height: 80px;
  border-top-color: var(--accent);
}

.loading-ring:nth-child(2) {
  width: 60px;
  height: 60px;
  top: 10px;
  left: 10px;
  border-right-color: var(--amber);
  animation-delay: 0.2s;
}

.loading-ring:nth-child(3) {
  width: 40px;
  height: 40px;
  top: 20px;
  left: 20px;
  border-bottom-color: var(--text-muted);
  animation-delay: 0.4s;
}

@keyframes ring-rotate {
  to { transform: rotate(360deg); }
}

.loading-text,
.waiting-text {
  font-size: 0.88rem;
  font-family: var(--font-serif);
  color: var(--text-primary);
  margin: 0 0 8px;
}

.waiting-hint {
  font-size: 0.78rem;
  color: var(--text-muted);
  margin: 0;
}

.waiting-icon {
  margin-bottom: 20px;
}

.network-icon {
  width: 100px;
  height: 100px;
  opacity: 0.3;
}
.network-icon circle, .network-icon line {
  stroke: var(--text-muted);
}

.graph-view {
  width: 100%;
  height: 100%;
  position: relative;
}

.graph-svg {
  width: 100%;
  height: 100%;
  display: block;
}

.graph-building-hint {
  position: absolute;
  bottom: 16px;
  left: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: var(--accent-bg);
  border: 1px solid var(--accent-border);
  border-radius: var(--radius-sm);
  font-size: 0.78rem;
  color: var(--accent-dark);
}

.building-dot {
  width: 7px;
  height: 7px;
  background: var(--accent);
  border-radius: 50%;
  animation: pulse 1.5s infinite;
}

/* Node/Edge Detail Panel */
.detail-panel {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 320px;
  max-height: calc(100% - 32px);
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  z-index: 100;
}

.detail-panel-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: var(--bg-warm);
  border-bottom: 1px solid var(--border);
}

.detail-title {
  font-size: 0.88rem;
  font-weight: 600;
  font-family: var(--font-serif);
  color: var(--text-primary);
}

.detail-badge {
  padding: 2px 10px;
  font-size: 0.72rem;
  color: var(--text-inverse);
  border-radius: var(--radius-sm);
}

.detail-close {
  margin-left: auto;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  font-size: 1.2rem;
  color: var(--text-muted);
  cursor: pointer;
  transition: color 0.2s;
}

.detail-close:hover {
  color: var(--text-primary);
}

.detail-content {
  padding: 16px;
  overflow-y: auto;
  flex: 1;
}

.detail-row {
  display: flex;
  align-items: flex-start;
  margin-bottom: 12px;
}

.detail-label {
  font-size: 0.78rem;
  color: var(--text-muted);
  min-width: 70px;
  flex-shrink: 0;
}

.detail-value {
  font-size: 0.82rem;
  color: var(--text-secondary);
  word-break: break-word;
}

.detail-value.uuid {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  color: var(--text-muted);
}

.detail-section {
  margin-bottom: 12px;
}

.detail-summary {
  margin: 8px 0 0 0;
  font-size: 0.82rem;
  color: var(--text-secondary);
  line-height: 1.6;
  padding: 10px;
  background: var(--bg-elevated);
  border-left: 3px solid var(--accent);
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
}

.detail-labels {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.label-tag {
  padding: 2px 8px;
  font-size: 0.72rem;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  color: var(--text-secondary);
  border-radius: var(--radius-sm);
}

/* Edge Detail Relationship Display */
.edge-relation {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 16px;
  padding: 12px;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.edge-source,
.edge-target {
  font-size: 0.82rem;
  font-weight: 500;
  color: var(--text-primary);
}

.edge-arrow {
  color: var(--text-muted);
}

.edge-type {
  padding: 2px 8px;
  font-size: 0.72rem;
  background: var(--text-primary);
  color: var(--text-inverse);
  border-radius: var(--radius-sm);
}

.detail-value.highlight {
  font-weight: 600;
  color: var(--text-primary);
}

.detail-subtitle {
  font-size: 0.88rem;
  font-weight: 600;
  font-family: var(--font-serif);
  color: var(--text-primary);
  margin: 16px 0 12px 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
}

/* Properties List */
.properties-list {
  margin-top: 8px;
  padding: 10px;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.property-item {
  display: flex;
  margin-bottom: 6px;
  font-size: 0.82rem;
}

.property-item:last-child {
  margin-bottom: 0;
}

.property-key {
  color: var(--text-muted);
  margin-right: 8px;
  font-family: var(--font-mono);
}

.property-value {
  color: var(--text-secondary);
  word-break: break-word;
}

/* Episodes List */
.episodes-list {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.episode-tag {
  display: block;
  padding: 6px 10px;
  font-size: 0.72rem;
  font-family: var(--font-mono);
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  color: var(--text-muted);
  word-break: break-all;
  border-radius: var(--radius-sm);
}

.error-icon {
  font-size: 2rem;
  display: block;
  margin-bottom: 10px;
}

/* Graph Legend */
.graph-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  padding: 10px 20px;
  border-top: 1px solid var(--border);
  background: var(--bg-surface);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.72rem;
}

.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.legend-label {
  color: var(--text-secondary);
}

.legend-count {
  color: var(--text-muted);
}

/* Right Panel */
.right-panel {
  width: 50%;
  flex: none;
  display: flex;
  flex-direction: column;
  background: var(--bg-warm);
  transition: width 0.35s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.3s ease, transform 0.3s ease;
  overflow: hidden;
  opacity: 1;
}

.right-panel.hidden {
  width: 0;
  opacity: 0;
  transform: translateX(20px);
  pointer-events: none;
}

.right-panel .panel-header.dark-header {
  background: var(--bg-surface);
  color: var(--text-primary);
  border-bottom: 1px solid var(--border);
}

.right-panel .header-icon {
  color: var(--accent);
  margin-right: 8px;
}

/* Workflow Content */
.process-content {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
}

/* Workflow Phases */
.process-phase {
  margin-bottom: 20px;
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  opacity: 0.5;
  transition: all 0.3s;
  background: var(--bg-surface);
}

.process-phase.active,
.process-phase.completed {
  opacity: 1;
}

.process-phase.active {
  border-color: var(--accent-border);
  box-shadow: var(--shadow-sm);
}

.process-phase.completed {
  border-color: var(--emerald);
}

.phase-header {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  padding: 14px 16px;
  background: var(--bg-surface);
  border-bottom: 1px solid var(--border-light);
  border-radius: var(--radius-md) var(--radius-md) 0 0;
}

.process-phase.active .phase-header {
  background: var(--accent-bg);
}

.process-phase.completed .phase-header {
  background: rgba(58, 143, 110, 0.06);
}

.phase-num {
  font-size: 1.5rem;
  font-weight: 700;
  font-family: var(--font-serif);
  color: var(--text-muted);
  line-height: 1;
}

.process-phase.active .phase-num {
  color: var(--accent);
}

.process-phase.completed .phase-num {
  color: var(--emerald);
}

.phase-info {
  flex: 1;
}

.phase-title {
  font-size: 1rem;
  font-weight: 600;
  font-family: var(--font-serif);
  margin-bottom: 4px;
}

.phase-api {
  font-size: 0.72rem;
  color: var(--text-muted);
  font-family: var(--font-mono);
}

.phase-status {
  font-size: 0.72rem;
  padding: 4px 10px;
  background: var(--bg-elevated);
  color: var(--text-muted);
  border-radius: var(--radius-sm);
}

.phase-status.active {
  background: var(--text-primary);
  color: var(--text-inverse);
}

.phase-status.completed {
  background: var(--emerald);
  color: #fff;
}

/* Phase Details */
.phase-detail {
  padding: 16px;
}

/* Entity Tags */
.entity-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.entity-tag {
  font-size: 0.72rem;
  padding: 4px 10px;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  color: var(--text-secondary);
  border-radius: var(--radius-sm);
}

/* Relationship List */
.relation-list {
  font-size: 0.8rem;
}

.relation-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 0;
  border-bottom: 1px dashed var(--border-light);
}

.relation-item:last-child {
  border-bottom: none;
}

.rel-source,
.rel-target {
  color: var(--text-secondary);
}

.rel-arrow {
  color: var(--text-muted);
}

.rel-name {
  color: var(--accent-dark);
  font-weight: 500;
}

.relation-more {
  padding-top: 8px;
  color: var(--text-muted);
  font-size: 0.72rem;
}

/* Ontology Generation Progress */
.ontology-progress {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: var(--accent-bg);
  border: 1px solid var(--accent-border);
  border-radius: var(--radius-sm);
}

.progress-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid var(--border);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.progress-text {
  font-size: 0.82rem;
  color: var(--text-secondary);
}

/* Waiting State */
.waiting-state {
  padding: 16px;
  background: var(--bg-elevated);
  border: 1px dashed var(--border);
  text-align: center;
  border-radius: var(--radius-sm);
}

.waiting-hint {
  font-size: 0.85rem;
  color: var(--text-muted);
}

/* Progress Bar */
.progress-bar {
  height: 6px;
  background: var(--bg-elevated);
  margin-bottom: 8px;
  overflow: hidden;
  border-radius: 3px;
}

.progress-fill {
  height: 100%;
  background: var(--accent);
  transition: width 0.3s;
  border-radius: 3px;
}

.progress-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.72rem;
}

.progress-message {
  color: var(--text-muted);
}

.progress-percent {
  color: var(--accent-dark);
  font-weight: 600;
}

/* Build Results */
.build-result {
  display: flex;
  gap: 16px;
}

.result-item {
  flex: 1;
  text-align: center;
  padding: 12px;
  background: var(--bg-elevated);
  border-radius: var(--radius-sm);
  border: 1px solid var(--border);
}

.result-value {
  display: block;
  font-size: 1.5rem;
  font-weight: 700;
  font-family: var(--font-serif);
  color: var(--text-primary);
  margin-bottom: 4px;
}

.result-label {
  font-size: 0.68rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* Next Step Button */
.next-step-section {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--border);
}

.next-step-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 14px;
  background: var(--text-primary);
  color: var(--text-inverse);
  border: none;
  border-radius: var(--radius-md);
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s;
  box-shadow: var(--shadow-sm);
}

.next-step-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.next-step-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
}

.btn-arrow {
  font-size: 1.2rem;
}

/* Project Info Panel */
.project-panel {
  border-top: 1px solid var(--border);
  background: var(--bg-surface);
}

.project-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 20px;
  border-bottom: 1px solid var(--border);
}

.project-icon {
  color: var(--accent);
}

.project-title {
  font-size: 0.82rem;
  font-weight: 600;
  font-family: var(--font-serif);
}

.project-details {
  padding: 14px 20px;
}

.project-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 8px 0;
  border-bottom: 1px dashed var(--border-light);
  font-size: 0.78rem;
}

.project-item:last-child {
  border-bottom: none;
}

.item-label {
  color: var(--text-muted);
  flex-shrink: 0;
}

.item-value {
  color: var(--text-secondary);
  text-align: right;
  max-width: 60%;
  word-break: break-all;
}

.item-value.code {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  color: var(--text-muted);
}

/* Responsive */
@media (max-width: 1024px) {
  .main-content {
    flex-direction: column;
  }
  
  .left-panel {
    width: 100% !important;
    border-right: none;
    border-bottom: 1px solid var(--border);
    height: 50vh;
  }
  
  .right-panel {
    width: 100% !important;
    height: 50vh;
    opacity: 1 !important;
    transform: none !important;
  }
  
  .right-panel.hidden {
      display: none;
  }
}
</style>