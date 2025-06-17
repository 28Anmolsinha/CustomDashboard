
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
        <div class="control-card">
          <h2 class="card-title">Options</h2>
          <ul class="options-list-vertical">
            <li
              v-for="(option, idx) in options"
              :key="idx"
              class="option-card"
              :style="{ backgroundColor: optionColorMap[option], color: isDarkMode ? '#fff' : '#000' }"
              draggable="true"
              @dragstart="onDragStart(option, $event)"
            >
              {{ option }}
            </li>
          </ul>
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

        <!-- Chart Layout -->
        <div class="control-card">
          <h2 class="card-title">Chart Layout</h2>
          <label><input type="radio" value="full" v-model="chartLayout" /> Full Width</label>
          <label><input type="radio" value="half" v-model="chartLayout" /> Side by Side</label>
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
    // Static options
    options: ['Total Call Initiated', 'Call Dailed Out', 'Answered Calls', 'Unanswered Calls','Failed Calls','Dropped Calls', 'Busy Calls','No Answers','Invalid Number'],
  optionColorMap: {
  'Total Call Initiated': '#34d399',
  'Call Dailed Out': '#60a5fa',
  'Answered Calls': '#fbbf24',
  'Unanswered Calls': '#f87171',
  'Failed Calls': '#c084fc',          
  'Dropped Calls': '#f472b6',         
  'Busy Calls': '#fb923c',            
  'No Answers': '#818cf8',            
  'Invalid Number': '#A52A2A'         
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

    // 🧩 Drop zones with move support
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
],availableSlots: [
  { top: 20, left: 20 },
  { top: 20, left: 590 },
  { top: 20, left: 1160 },
  { top: 220, left: 20 },
  { top: 220, left: 590 },
  { top: 220, left: 1160 }
],
    
    //showSecondZone: false,
 
    // 🖱️ Zone moving
    draggingZoneIndex: null,
    dragOffsetX: 0,
    dragOffsetY: 0,

    // Widget resizing
    resizingIndex: null,
    resizingZone: null,
    startX: 0,
    startY: 0,
    startWidth: 0,
    startHeight: 0

    
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
      droppedOptions: zone.droppedOptions || [],
      droppedColors: zone.droppedColors || [],
      itemWidths: zone.itemWidths || [],
      itemHeights: zone.itemHeights || [],
      showGraph: zone.showGraph || false,
      top: zone.top || 0,
      left: zone.left || 0,
      chartType: zone.chartType || 'bar',
      timeInterval: zone.timeInterval || 'daywise',
      chartData: zone.chartData || null // Save chart data
    })),
    chartLayout: this.chartLayout,
    isDarkMode: this.isDarkMode
  };
  localStorage.setItem('savedDashboard', JSON.stringify(state));
  alert('Dashboard saved successfully!');
},

loadDashboard() {
  const saved = localStorage.getItem('savedDashboard');
  if (saved) {
    try {
      const state = JSON.parse(saved);
      
   
      this.zones = [];
      state.zones.forEach((zone) => {
        this.zones.push({
          droppedOptions: zone.droppedOptions || [],
          droppedColors: zone.droppedColors || [],
          itemWidths: zone.itemWidths || [],
          itemHeights: zone.itemHeights || [],
          showGraph: zone.showGraph || false,
          top: zone.top || 0,
          left: zone.left || 0,
          chartType: zone.chartType || 'bar',
          timeInterval: zone.timeInterval || 'daywise',
          chartData: zone.chartData || null
        });
      });

      // Restore chart layout and dark mode
      this.chartLayout = state.chartLayout || 'full';
      this.isDarkMode = state.isDarkMode ?? false;

      alert('Dashboard loaded successfully!');
    } catch (e) {
      alert('Error loading dashboard: ' + e.message);
    }
  } else {
    alert('No saved dashboard found.');
  }
},
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

  const containerHeight = window.innerHeight - 100; // Adjust if you have a header/footer

  // Try filling vertically first
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

  // Optional: prevent overlapping with other zones
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
/* .add-zone-btn {
  margin-top: 10px;
  padding: 8px;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
} */
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

.add-zone-btn {

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


.app-container.dark .add-zone-btn{
   background-color: #38b2ac;
}
.add-zone-btn:hover {
  background-color: #276749;
}

.resizable-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 4px 8px;
  box-sizing: border-box;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  border-radius: 6px;
  font-size: 0.85rem;
  position: relative;
}
.dropped-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  position: relative;
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
.dark-toggle {
  padding: 6px 14px;
  background-color: #2f855a;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  width: 100%;
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
.app-container.dark .left-drawer {
  background-color: #2d3748;
  border-color: #4fd1c5;
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

.control-card {
  background-color: #d9f0d9;
  border: 1px solid #68d391;
  border-radius: 8px;
  padding: 16px 24px;
}
.app-container.dark .control-card {
  background-color: #2a4365;
  border-color: #4fd1c5;
}
.card-title {
  margin-bottom: 12px;
  font-size: 1.1rem;
  font-weight: 600;
  color: #276749;
}
.app-container.dark .card-title {
  color: #9ae6b4;
}
.control-card label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #276749;
}
.app-container.dark .control-card label {
  color: #9ae6b4;
}
.control-card input[type='radio'] {
  margin-right: 8px;
}

.drag-drop-zone {
  margin: 0;
  width: 100%;
  max-width: 100%;
  min-height: 250px;
  border: 2px dashed #68d391;
  border-radius: 8px;
  background-color: #f0fff4;
  position: relative;
  padding: 16px;
  box-sizing: border-box;
}
.app-container.dark .drag-drop-zone {
  border-color: #4fd1c5;
  background-color: #2d3748;
}
.drop-placeholder {
  color: #276749;
  font-size: 1rem;
  font-weight: 500;
  text-align: center;
  padding: 80px 0;
  opacity: 0.7;
}
.app-container.dark .drop-placeholder {
  color: #9ae6b4;
}

.options-list-vertical {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.option-card {
  background-color: #9ae6b4; 
  border-radius: 6px;
  padding: 10px 14px;
  font-weight: 600;
  text-align: center;
  cursor: grab;
  transition: transform 0.2s;
}
.option-card:hover {
  transform: translateY(-2px);
}
.app-container.dark .option-card {
  background-color: #38b2ac;
  color: #fff;
}

.dropped-list {
  display: flex;
  gap: 10px;
  list-style: none;
  padding: 0;
  flex-wrap: wrap;
  position: relative;
}
.dropped-list li {
  background-color: #68d391;
  border-radius: 6px;
  font-weight: 500;
  color: #f0fff4;
  position: relative;
  user-select: none;
  box-sizing: border-box;
}
.app-container.dark .dropped-list li {
  background-color: #4fd1c5;
  color: #1a202c;
}

.resizable-item {
  position: relative;
  padding: 6px 28px 6px 10px; /* leave space for ✖ */
  box-sizing: border-box;
  border-radius: 6px;
  font-size: 0.85rem;
  color: #fff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
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
.show-graph-btn {

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

.app-container.dark .show-graph-btn{
   background-color: #38b2ac;
}
.show-graph-btn:hover {
  background-color: #276749;
}

.save-dashboard {
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

.app-container.dark .save-dashboard{
   background-color: #38b2ac;
}

.save-dashboard :hover {
  background-color: #276749;
}

.load-dashboard {
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
.app-container.dark .load-dashboard{
   background-color: #38b2ac;
}
.load-dashboard :hover {
  background-color: #276749;
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
.app-container.dark .graph-box {
  background-color: #2d3748;
  border-color: #4fd1c5;
}

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
.app-container.dark .combined-zone {
  background-color: #2d3748;
  border-color: #4fd1c5;
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
</style>
