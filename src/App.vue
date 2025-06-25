<template>
  <div :class="['app-container', { dark: isDarkMode }]">
    <!-- LEFT: Drawer -->
    <div class="left-drawer" :class="{ open: isDrawerOpen }" @click.self="toggleDrawer">
      <div v-if="isDrawerOpen" class="drawer-content">
        <button class="close-drawer-btn" @click="toggleDrawer">✖ </button>
        <!-- Dark Mode Toggle -->
        <button class="dark-toggle" @click="isDarkMode = !isDarkMode">
          <span class="toggle-icon">{{ isDarkMode ? '🌞' : '🌙' }}</span>
          {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
        </button>

        <!-- Draggable Options -->
      <!-- Draggable Options -->
<div class="control-card">
  <h2 class="card-title">Options</h2>
  
  <!-- Call Metrics Dropdown -->
  <div class="widget-dropdown-container" style="margin-bottom: 10px;">
    <button class="dropdown-toggle" @click="showCallOptions = !showCallOptions">
      Call Metrics 
      <span class="dropdown-icon" :class="{ rotated: showCallOptions }">▼</span>
    </button>
    <transition name="fade">
      <div v-if="showCallOptions" class="dropdown-options-container">
        <ul class="options-list-vertical">
          <li
            v-for="(option, idx) in callOptions"
            :key="'call-'+idx"
            class="option-card"
            :style="{ backgroundColor: optionColorMap[option], color: isDarkMode ? '#fff' : '#000' }"
            draggable="true"
            @dragstart="onDragStart(option, $event)"
          >
            {{ option }}
          </li>
        </ul>
      </div>
    </transition>
  </div>

  <!-- Performance Metrics Dropdown -->
  <div class="widget-dropdown-container">
    <button class="dropdown-toggle" @click="showPerfOptions = !showPerfOptions">
      Performance Metrics 
     <span class="dropdown-icon" :class="{ rotated: showPerfOptions }">▼</span>
    </button>
    <transition name="fade">
      <div v-if="showPerfOptions" class="dropdown-options-container">
        <ul class="options-list-vertical">
          <li
            v-for="(option, idx) in perfOptions"
            :key="'perf-'+idx"
            class="option-card"
            :style="{ backgroundColor: optionColorMap[option], color: isDarkMode ? '#fff' : '#000' }"
            draggable="true"
            @dragstart="onDragStart(option, $event)"
          >
            {{ option }}
          </li>
        </ul>
      </div>
    </transition>
  </div>

  <!-- Campaign Metrics Dropdown -->
  <div class="widget-dropdown-container">
    <button class="dropdown-toggle" @click="showcampaign = !showcampaign">
      Campaign Metrics 
          <span class="dropdown-icon" :class="{ rotated: showcampaign }">▼</span>
    </button>
    <transition name="fade">
      <div v-if="showcampaign" class="dropdown-options-container">
        <ul class="options-list-vertical">
          <li
            v-for="(option, idx) in campaignOptions"
            :key="'perf-'+idx"
            class="option-card"
            :style="{ backgroundColor: optionColorMap[option], color: isDarkMode ? '#fff' : '#000' }"
            draggable="true"
            @dragstart="onDragStart(option, $event)"
          >
            {{ option }}
          </li>
        </ul>
      </div>
    </transition>
  </div>

    <!-- Quality Metrics Dropdown -->
  <div class="widget-dropdown-container">
    <button class="dropdown-toggle" @click="qualitymetrices = !qualitymetrices">
      Quality Metrics 
         <span class="dropdown-icon" :class="{ rotated: qualitymetrices }">▼</span>
    </button>
    <transition name="fade">
      <div v-if="qualitymetrices" class="dropdown-options-container">
        <ul class="options-list-vertical">
          <li
            v-for="(option, idx) in qualityOptions"
            :key="'perf-'+idx"
            class="option-card"
            :style="{ backgroundColor: optionColorMap[option], color: isDarkMode ? '#fff' : '#000' }"
            draggable="true"
            @dragstart="onDragStart(option, $event)"
          >
            {{ option }}
          </li>
        </ul>
      </div>
    </transition>
  </div>
</div>

        <!-- Button -->
        <div class="control-card">
          <h2 class="card-title">Buttons</h2>
          <button
  class="show-graph-btn"
  :disabled="zones.every(zone => zone.droppedOptions.length === 0)"
  @click="showGraphsForZones"
>
  Show Graph
</button>

  <button
    class="add-zone-btn"
    :disabled="zones.length >= 6"
    @click="addDropZone"
  >
    Add Drop Zone
  </button>
        </div>
        <div class="control-card">
  <h2 class="card-title">Dashboard</h2>
  <button class="save-dashboard" @click="saveDashboard">💾 Save Dashboard</button>
  <button class="load-dashboard" @click="loadDashboard">🔁 Load Dashboard</button>
</div>
      </div>
    </div>

    <!-- RIGHT: Unified Zone -->
<div class="right-content">
 <div
  class="combined-zone"
  v-for="(zone, zIdx) in visibleZones"
  :key="'zone-' + zIdx"
  @dragover.prevent
  @drop="onDrop($event, zIdx)"
  @mousedown="startZoneDrag($event, zIdx)"
  :style="{
    top: zone.top + 'px',
    left: zone.left + 'px',
    position: 'absolute'
  }"
>
 <!-- Delete Button -->
  <button class="delete-zone-btn" @click="confirmDeleteZone(zIdx)">✖</button>

    <div v-if="!zone.droppedOptions.length && !zone.showGraph" class="drop-placeholder">
      Drag and Drop your widgets into this zone.
    </div>

    <ul class="dropped-list">
      <li
        v-for="(opt, i) in zone.droppedOptions"
        :key="i"
        class="resizable-item"
        :style="{
          position: 'absolute',
          top: '10px',
          top: Math.floor(i / 9) * 45 + 10 + 'px', 
          left: (i % 9) * 90 + 10 + 'px',
          width: zone.itemWidths[i] + 'px',
          height: zone.itemHeights[i] + 'px',
          backgroundColor: zone.droppedColors[i]
        }"
      >
        {{ opt }}
        <span class="remove-btn" @click="removeOption(zIdx, i)">✖</span>
        <div class="resize-handle" @mousedown.prevent="initResize($event, zIdx, i)"></div>
      </li>
    </ul>

    <div v-if="zone.showGraph" class="charts-wrapper">
      
      <div v-if="chartLayout === 'half'" class="charts-container half-layout">
        <ChartCard
          v-for="(chart, index) in 2"
          :key="index"
          :chartType="zone.chartType"
          :timeInterval="zone.timeInterval"
          :droppedOptions="zone.droppedOptions"
          :droppedColors="zone.droppedColors"
          :graph-data="zone.graphData"
          :savedData="zone.chartData"
          @updateChartType="zone.chartType = val"
          @updateTimeInterval="zone.timeInterval = val"
          @chartDataGenerated="(data) => zone.chartData = data"
          
        />
      </div>
      <div v-else class="charts-container full-layout">
        <ChartCard
          :chartType="zone.chartType"
          :timeInterval="zone.timeInterval"
          :droppedOptions="zone.droppedOptions"
          :droppedColors="zone.droppedColors"
          :graph-data="zone.graphData"
          :savedData="zone.chartData" 
         @updateChartType="val => zone.chartType = val"
         @updateTimeInterval="val => zone.timeInterval = val"
         @chartDataGenerated="(data) => zone.chartData = data"
        />
      </div>
    </div>
  </div>
</div>
  </div>
</template>

<script>
import ChartCard from './components/ChartCard.vue';

export default {
  name: 'App',
  components: { ChartCard },
data() {
  return {
    showCallOptions: false,
    showPerfOptions: false,
    showcampaign: false,
    qualitymetrices:false,

    callOptions: [
      'Total Call Initiated', 
      'Call Dailed Out', 
      'Answered Calls',
      'Unanswered Calls',
      'Failed Calls',
      'Dropped Calls',
      'Busy Calls',
      'No Answers',
      'Invalid Number'
    ],
    perfOptions: [
      'Answer Rate (%)',
      'Connection Rate (%)',
      'Success Rate (%)',
      'Failure Rate (%)',
      'Average Call Duration',
      'Average Hold Time',
      'Average Time to Connect'
    ],
    campaignOptions: [
      'Campaign Executed',
      'Campaign Reached',
      'DND (%)',
      'DTMF Inputs Captured',
      'DTMF Input Distribution',
      'Follow-up Action Rate'
    ],
    qualityOptions:[
      'Retry Attempts',
      'CLI Quality'
    ],

    // Combined color map for all options
   optionColorMap: {
  // Call Metrics Colors
  'Total Call Initiated': '#16a34a',     
  'Call Dailed Out': '#2563eb',         
  'Answered Calls': '#facc15',           
  'Unanswered Calls': '#ef4444',         
  'Failed Calls': '#a855f7',             
  'Dropped Calls': '#ec4899',            
  'Busy Calls': '#f97316',               
  'No Answers': '#6366f1',               
  'Invalid Number': '#7c3aed',           

  // Performance Metrics Colors
  'Answer Rate (%)': '#10b981',        
  'Connection Rate (%)': '#3b82f6',      
  'Success Rate (%)': '#8b5cf6',         
  'Failure Rate (%)': '#e11d48',         
  'Average Call Duration': '#fbbf24',    
  'Average Hold Time': '#f97316',        
  'Average Time to Connect': '#06b6d4',  

  // Campaign Metrics Colors
  'Campaign Executed': '#22c55e',    
  'Campaign Reached': '#0ea5e9',       
  'DND (%)': '#dc2626',                 
  'DTMF Inputs Captured': '#eab308',   
  'DTMF Input Distribution': '#ea580c',  
  'Follow-up Action Rate': '#9333ea',    

  // Quality Metrics Colors
  'Retry Attempts': '#be123c',          
  'CLI Quality': '#7c3aed'               
},

    // Chart controls 
    timeInterval: 'daywise',
    chartType: 'bar',
    chartLayout: 'full',

    // UI controls 
    draggedOption: null,
    isDrawerOpen: false,
    isDarkMode: false,

    // Default sizes for resizable items 
    lastSafeWidth: 300,
    lastSafeHeight: 600,

    // Drop zones with move support 
    zones: [
      {
        droppedOptions: [],
        droppedColors: [],
        itemWidths: [],
        itemHeights: [],
        showGraph: false,
        top: 100,
        left: 100,
        chartType: 'bar',
        timeInterval: 'daywise'
      }
    ],
    availableSlots: [
      { top: 20, left: 20 },
      { top: 20, left: 590 },
      { top: 20, left: 1160 },
      { top: 220, left: 20 },
      { top: 220, left: 590 },
      { top: 220, left: 1160 }
    ],
    
    // Zone moving 
    draggingZoneIndex: null,
    dragOffsetX: 0,
    dragOffsetY: 0,

    // Widget resizing
    resizingIndex: null,
    resizingZone: null,
    startX: 0,
    startY: 0,
    startWidth: 0,
    startHeight: 0,

    // Store dashboard Json 
    saveDashboardJson: ''
  };
},
computed: {
  visibleZones() {
    return this.zones;
  }
},
  methods: {
  saveDashboard() {
  const state = {
    zones: this.zones.map(zone => ({
      chartData: zone.chartData || null,
      showGraph: zone.showGraph || false,
      top: zone.top || 0,
      left: zone.left || 0,
      chartType: zone.chartType || 'bar',
      timeInterval: zone.timeInterval || 'daywise'
    })),
    chartLayout: this.chartLayout,
    isDarkMode: this.isDarkMode
  };

  const jsonState = JSON.stringify(state);
  localStorage.setItem('savedDashboard', jsonState);
  this.saveDashboardJson = jsonState;

  console.log('Saved Dashboard as JSON string:', jsonState);
  alert('Dashboard saved successfully!');
}
,

loadDashboard() {
  const saved = localStorage.getItem('savedDashboard');
  if (saved) {
    try {
      const state = JSON.parse(saved);
      this.zones = [];

      state.zones.forEach((zone) => {
        const datasets = zone.chartData?.datasets || [];

        this.zones.push({
          chartData: zone.chartData || null,
          showGraph: zone.showGraph || false,
          top: zone.top || 0,
          left: zone.left || 0,
          chartType: zone.chartType || 'bar',
          timeInterval: zone.timeInterval || 'daywise',

       
          droppedOptions: datasets.map(ds => ds.label),
          droppedColors: datasets.map(ds => ds.backgroundColor),
          itemWidths: datasets.map(() => 80),     
          itemHeights: datasets.map(() => 35)
        });
      });

      this.chartLayout = state.chartLayout || 'full';
      this.isDarkMode = state.isDarkMode ?? false;

      alert('Dashboard loaded successfully!');
    } catch (e) {
      alert('Error loading dashboard: ' + e.message);
    }
  } else {
    alert('No saved dashboard found.');
  }
}
,
    confirmDeleteZone(zoneIndex) {
  const confirmed = window.confirm("Do you really want to delete this drop zone?");
  if (confirmed) {
    this.zones.splice(zoneIndex, 1);
  }
},
addDropZone() {
  const zoneWidth = 320;
  const zoneHeight = 300;
  const padding = 20;

  const containerHeight = window.innerHeight - 100; 

  let found = false;
  let newTop = 0;
  let newLeft = 0;

  for (let col = 0; col < 10 && !found; col++) {
    for (let row = 0; row < Math.floor(containerHeight / (zoneHeight + padding)) && !found; row++) {
      const tryTop = row * (zoneHeight + padding);
      const tryLeft = col * (zoneWidth + padding);

      const isOverlapping = this.zones.some(zone => {
        const existingTop = zone.top;
        const existingLeft = zone.left;
        const existingRight = existingLeft + zoneWidth;
        const existingBottom = existingTop + zoneHeight;

        const newRight = tryLeft + zoneWidth;
        const newBottom = tryTop + zoneHeight;

        return !(newRight < existingLeft || tryLeft > existingRight ||
                 newBottom < existingTop || tryTop > existingBottom);
      });

      if (!isOverlapping) {
        newTop = tryTop;
        newLeft = tryLeft;
        found = true;
      }
    }
  }

  if (!found) {
    alert("No space left to add a new drop zone without overlapping.");
    return;
  }

  this.zones.push({
    droppedOptions: [],
    droppedColors: [],
    itemWidths: [],
    itemHeights: [],
    showGraph: false,
    top: newTop,
    left: newLeft,
    chartType: 'bar',
    timeInterval: 'daywise'
  });
}


,
startZoneDrag(event, zoneIndex) {
  // Prevent drag if user clicked on widget or chart area
  const isInsideItem = event.target.closest('.resizable-item');
  const isInsideChart = event.target.closest('.charts-wrapper');
  if (isInsideItem || isInsideChart) return;

  const zone = this.zones[zoneIndex];

  this.draggingZoneIndex = zoneIndex;
  this.dragOffsetX = event.clientX - zone.left;
  this.dragOffsetY = event.clientY - zone.top;

  window.addEventListener('mousemove', this.onZoneDrag);
  window.addEventListener('mouseup', this.stopZoneDrag);
},

onZoneDrag(event) {
  if (this.draggingZoneIndex === null) return;

  const zone = this.zones[this.draggingZoneIndex];
  //const rightContent = this.$el.querySelector('.right-content');
  const containerRect = document.body.getBoundingClientRect();
  const zoneWidth = 320;
  const zoneHeight = 300;

  let newLeft = event.clientX - this.dragOffsetX;
  let newTop = event.clientY - this.dragOffsetY;

  // Clamp within right-content container
  newLeft = Math.max(0, Math.min(newLeft, window.innerWidth - zoneWidth));
  newTop = Math.max(0, Math.min(newTop, containerRect.height - zoneHeight));


  if (!this.willOverlap(newLeft, newTop, this.draggingZoneIndex)) {
    this.$set(zone, 'left', newLeft);
    this.$set(zone, 'top', newTop);
  }
},

stopZoneDrag() {
  window.removeEventListener('mousemove', this.onZoneDrag);
  window.removeEventListener('mouseup', this.stopZoneDrag);
  this.draggingZoneIndex = null;
},

willOverlap(newLeft, newTop, movingIndex) {
  // const movingZone = this.zones[movingIndex];
  const width = 300;
  const height = 300;

  return this.zones.some((zone, idx) => {
    if (idx === movingIndex) return false;

    const otherLeft = zone.left;
    const otherTop = zone.top;

    return !(
      newLeft + width < otherLeft ||
      newLeft > otherLeft + width ||
      newTop + height < otherTop ||
      newTop > otherTop + height
    );
  });
},

    showGraphsForZones() {
  this.zones.forEach(zone => {
    zone.showGraph = zone.droppedOptions.length > 0;
  });
},
    toggleDrawer() {
      this.isDrawerOpen = !this.isDrawerOpen;
    },
    onDragStart(option, event) {
      event.dataTransfer.setData('type', 'option');
      event.dataTransfer.setData('option', option);
    },
onDrop(event, zoneIndex) {
  const type = event.dataTransfer.getData('type');
  if (type === 'option') {
    const opt = event.dataTransfer.getData('option');
    const zone = this.zones[zoneIndex];
    if (opt && !zone.droppedOptions.includes(opt)) {
      zone.droppedOptions.push(opt);
      const dropIndex = zone.droppedOptions.length - 1;
      const color = this.optionColorMap[opt] || '#a78bfa';
      zone.droppedColors.push(color);


     
      this.$set(zone.itemWidths, dropIndex, 80);
      this.$set(zone.itemHeights, dropIndex, 35);
    }
  }
},
  removeOption(zoneIndex, index) {
  const zone = this.zones[zoneIndex];
  zone.droppedOptions.splice(index, 1);
  zone.droppedColors.splice(index, 1);
  zone.itemWidths.splice(index, 1);
  zone.itemHeights.splice(index, 1);
}

,
initResize(event, zoneIndex, index) {
  this.resizingIndex = index;
  this.resizingZone = zoneIndex;
  this.startX = event.clientX;
  this.startY = event.clientY;
  const zone = this.zones[zoneIndex];
  this.startWidth = zone.itemWidths[index] || 100;
  this.startHeight = zone.itemHeights[index] || 40;
  window.addEventListener('mousemove', this.resizeItem);
  window.addEventListener('mouseup', this.stopResize);
},

resizeItem(event) {
  if (this.resizingIndex === null || this.resizingZone === null) return;
  const dx = event.clientX - this.startX;
  const dy = event.clientY - this.startY;
  const newWidth = Math.max(50, this.startWidth + dx);
  const newHeight = Math.max(30, this.startHeight + dy);
  const zone = this.zones[this.resizingZone];
  const dropZone = this.$el.querySelectorAll('.combined-zone')[this.resizingZone];
  const itemEl = dropZone.querySelectorAll('.resizable-item')[this.resizingIndex];

  const dropZoneRect = dropZone.getBoundingClientRect();
  const itemRect = itemEl.getBoundingClientRect();

  const maxWidth = dropZoneRect.right - itemRect.left - 10;
  const maxHeight = dropZoneRect.bottom - itemRect.top - 10;

  const contentMinWidth = itemEl.scrollWidth + 16;
  const contentMinHeight = itemEl.scrollHeight + 16;

  const willOverflowX = newWidth > maxWidth || newWidth < contentMinWidth;
  const willOverflowY = newHeight > maxHeight || newHeight < contentMinHeight;

  if (!willOverflowX && !willOverflowY) {
    this.$set(zone.itemWidths, this.resizingIndex, newWidth);
    this.$set(zone.itemHeights, this.resizingIndex, newHeight);
  }
},

  stopResize() {
  window.removeEventListener('mousemove', this.resizeItem);
  window.removeEventListener('mouseup', this.stopResize);
  this.resizingIndex = null;
  this.resizingZone = null;
}

  },
  watch: {
    droppedOptions(newOptions) {
      newOptions.forEach((_, i) => {
        if (this.itemWidths[i] === undefined) this.$set(this.itemWidths, i, 100);
        if (this.itemHeights[i] === undefined) this.$set(this.itemHeights, i, 40);
      });
      if (this.itemWidths.length > newOptions.length) {
        this.itemWidths.splice(newOptions.length);
        this.itemHeights.splice(newOptions.length);
      }
    }
  }
};
</script>

<style scoped>
/* Dropdown styles */
.dropdown-icon {
  transition: transform 0.2s ease;
  font-size: 0.8em;
  margin-left: 8px;
}

.dropdown-icon.rotated {
  transform: rotate(180deg);
}

.dropdown-toggle {
  width: 100%;
  padding: 10px 16px;
  background-color: #38a169;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  text-align: left;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: space-between;
  transition: all 0.2s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  margin-bottom: 8px;
}

.dropdown-toggle:hover {
  background-color: #2f855a;
  transform: translateY(-1px);
}

.dropdown-toggle[aria-expanded="true"] .dropdown-icon {
  transform: rotate(180deg);
}

.dropdown-options-container {
  background: white;
  border-radius: 6px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  margin-top: 4px;
  overflow: hidden;
  transition: all 0.2s ease;
  border: 1px solid #e2e8f0;
  position: relative;
  width: 100%;
  max-height: 300px;
  overflow-y: auto;
  z-index: 10;
}

.option-card {
  padding: 10px 16px;
  margin: 4px;
  border-radius: 4px;
  font-weight: 500;
  cursor: grab;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
  background-color: #9ae6b4; 
  text-align: center;
}

.option-card:hover {
  transform: translateX(4px);
  box-shadow: 0 2px 4px rgba(0,0,0,0.15);
}

.option-card:active {
  cursor: grabbing;
}

.widget-dropdown-container {
  margin-bottom: 16px;
  position: relative;
  display: block;
  width: 100%;
}

.options-list-vertical {
  margin: 0;
  padding: 5px 0;
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* Button styles */
.add-zone-btn, .show-graph-btn, .save-dashboard, .load-dashboard {
  background-color: #38a169;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  font-weight: 600;
  cursor: pointer;
  width: 100%;
  box-shadow: 0 2px 6px rgba(44, 62, 80, 0.08);
  transition: background-color 0.2s;
  margin-top: 12px;
}

.add-zone-btn:hover, .show-graph-btn:hover, 
.save-dashboard:hover, .load-dashboard:hover {
  background-color: #276749;
}

.add-zone-btn:disabled {
  background-color: #9ca3af;
  cursor: not-allowed;
}

.close-drawer-btn {
  background-color: transparent;
  color: inherit;
  font-size: 18px;
  border: none;
  cursor: pointer;
  margin-bottom: 10px;
  align-self: flex-end;
}

.delete-zone-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  background-color: #ef4444;
  color: white;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  font-size: 14px;
  cursor: pointer;
  z-index: 10;
}

.dark-toggle {
  padding: 8px 16px;
  background-color: #2f855a;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 600;
  transition: background-color 0.3s ease;
}

.dark-toggle:hover {
  background-color: #276749;
}

.toggle-icon {
  font-size: 1.2rem;
  transition: transform 0.3s ease;
}

.dark-toggle:hover .toggle-icon {
  transform: rotate(20deg);
}

/* App container and layout styles */
.app-container {
  display: flex;
  min-height: 100vh;
  font-family: 'Segoe UI', sans-serif;
  background-color: #e6f4ea;
  color: #1a202c;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.app-container.dark {
  background-color: #1a202c;
  color: #edf2f7;
}

.left-drawer {
  width: 40px;
  transition: width 0.3s ease;
  overflow-x: hidden;
  background-color: #c6f6d5;
  border-right: 2px solid #68d391;
}

.left-drawer.open {
  width: 280px;
}

.drawer-content {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.right-content {
  flex: 1;
  padding: 24px;
  position: relative;
  height: 400px;
}

/* Card styles */
.control-card {
  background-color: #d9f0d9;
  border: 1px solid #68d391;
  border-radius: 8px;
  padding: 16px 24px;
  position: relative;
  overflow: visible;
}

.card-title {
  margin-bottom: 16px;
  font-size: 1.1rem;
  font-weight: 600;
  color: #2d3748;
  padding-bottom: 8px;
  border-bottom: 2px solid #e2e8f0;
}

.control-card label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #276749;
}

.control-card input[type='radio'] {
  margin-right: 8px;
}

/* Original Drop Zone styles */
.drag-drop-zone {
  width: 100%;
  min-height: 250px;
  border: 2px dashed #68d391;
  border-radius: 8px;
  background-color: #f0fff4;
  position: relative;
  padding: 16px;
  box-sizing: border-box;
  margin: 0;
}

.drop-placeholder {
  color: #276749;
  font-size: 1rem;
  font-weight: 500;
  text-align: center;
  padding: 80px 0;
  opacity: 0.7;
}

.dropped-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  position: relative;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.resizable-item {
  position: relative;
  padding: 6px 28px 6px 10px;
  box-sizing: border-box;
  border-radius: 6px;
  font-size: 0.85rem;
  color: #fff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  background-color: #68d391;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.resize-handle {
  width: 12px;
  height: 12px;
  background-color: #2f855a;
  border-radius: 2px;
  position: absolute;
  right: 2px;
  bottom: 2px;
  cursor: se-resize;
  user-select: none;
}

.remove-btn {
  position: absolute;
  top: 4px;
  right: 4px;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  width: 16px;
  height: 16px;
  font-size: 12px;
  line-height: 16px;
  text-align: center;
  cursor: pointer;
  color: #fff;
  z-index: 2;
}

.movable-first {
  z-index: 10;
  cursor: move;
}

/* Combined Zone styles */
.combined-zone {
  position: relative;
  resize: both;
  overflow: auto;
  min-width: 300px;
  min-height: 300px;
  border: 2px dashed #68d391;
  border-radius: 8px;
  background-color: #f0fff4;
  padding: 16px;
  box-sizing: border-box;
  max-width: 100%;
  max-height: 90vh;
  cursor: move;
  z-index: 10;
}

/* Chart container styles */
.charts-container {
  display: flex;
  margin-top: 32px;
  gap: 32px;
  max-width: 100%;
  max-height: 100vh;
  height: 100%;
  overflow-x: auto;
  box-sizing: border-box;
}

.charts-container.half-layout > * {
  width: 50%;
  box-sizing: border-box;
}

.charts-container.full-layout {
  flex-direction: column;
}

.charts-container.full-layout > * {
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
}

.side-by-side-container {
  display: flex;
  gap: 20px;
  height: 100%;
  height: 50vh;
  align-items: stretch;
}

.half-zone {
  width: 50%;
  position: relative;
  min-height: 300px;
}

.graph-box {
  width: 50%;
  height: 50vh;
  min-height: 300px;
  background-color: #f0fff4;
  border: 2px solid #cbd5e0;
  border-radius: 8px;
  padding: 16px;
  box-sizing: border-box;
  overflow: hidden;
  display: flex;
}

.charts-wrapper {
  margin-top: 20px;
  position: relative;
}

.close-graph {
  position: absolute;
  top: -10px;
  right: -10px;
  background-color: #e53e3e;
  padding: 6px 10px;
  border-radius: 50%;
  font-weight: bold;
  color: white;
  cursor: pointer;
  z-index: 100;
}

/* Dark mode styles */
.app-container.dark .dropdown-toggle {
  background-color: #38b2ac;
}

.app-container.dark .dropdown-toggle:hover {
  background-color: #2c7a7b;
}

.app-container.dark .dropdown-options-container {
  background: #2d3748;
  border-color: #4a5568;
}

.app-container.dark .option-card {
  background-color: #38b2ac;
  color: #fff;
}

.app-container.dark .left-drawer {
  background-color: #2d3748;
  border-color: #4fd1c5;
}

.app-container.dark .control-card {
  background-color: #2a4365;
  border-color: #4fd1c5;
}

.app-container.dark .card-title {
  color: #edf2f7;
  border-bottom-color: #4a5568;
}

.app-container.dark .control-card label {
  color: #9ae6b4;
}

.app-container.dark .drag-drop-zone,
.app-container.dark .combined-zone {
  border-color: #4fd1c5;
  background-color: #2d3748;
}

.app-container.dark .drop-placeholder {
  color: #9ae6b4;
}

.app-container.dark .dropped-list li {
  background-color: #4fd1c5;
  color: #1a202c;
}

.app-container.dark .graph-box {
  background-color: #2d3748;
  border-color: #4fd1c5;
}

.app-container.dark .add-zone-btn,
.app-container.dark .show-graph-btn,
.app-container.dark .save-dashboard,
.app-container.dark .load-dashboard {
  background-color: #38b2ac;
}
</style>
