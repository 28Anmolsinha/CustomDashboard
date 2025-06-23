<template>
  <div
    :class="['chart-preview-card', { 'pie-small': chartType === 'pie' }]"
    v-if="droppedOptions.length"
  >
    <div class="chart-header">
      <h3 class="chart-heading">
        {{ capitalizedChartType }} – {{ capitalizedTimeInterval }}
      </h3>
      <div class="chart-controls">
        <div class="dropdown-group">
          <label>
            Chart Type:
            <select :value="chartType" @change="$emit('updateChartType', $event.target.value)">
              <option value="bar">Bar</option>
              <option value="line">Line</option>
              <option value="pie">Pie</option>
            </select>
          </label>
        </div>
        <div class="dropdown-group">
          <label>
            Time Interval:
            <select :value="timeInterval" @change="$emit('updateTimeInterval', $event.target.value)">
              <option value="daywise">Daywise</option>
              <option value="hourly">Hourly</option>
            </select>
          </label>
        </div>
      </div>
    </div>
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script>
import Chart from 'chart.js';

export default {
  name: 'ChartCard',
  props: {
    droppedOptions: { type: Array, default: () => [] },
    droppedColors: { type: Array, default: () => [] },
    chartType: { type: String, default: 'bar' },
    timeInterval: { type: String, default: 'daywise' }
  },
  computed: {
    capitalizedChartType() {
      return this.chartType.charAt(0).toUpperCase() + this.chartType.slice(1);
    },
    capitalizedTimeInterval() {
      return this.timeInterval === 'daywise' ? 'Daywise' : 'Hourly';
    }
  },
  watch: {
    chartType: 'updateChart',
    timeInterval: 'updateChart',
    droppedOptions: {
      handler: 'updateChart',
      deep: true
    },
    droppedColors: {
      handler: 'updateChart',
      deep: true
    }
  },
  mounted() {
    if (this.droppedOptions.length) this.renderChart();
  },
  methods: {
    renderChart() {
      const ctx = this.$refs.chartCanvas.getContext('2d');
      if (this.chartInstance) this.chartInstance.destroy();

      const chartData = this.getChartData(); 

      const isPie = this.chartType === 'pie';
      const config = {
        type: this.chartType,
        data: this.getChartData(),
        options: {
          responsive: true,
          maintainAspectRatio: isPie,
          aspectRatio: isPie ? 1 : undefined,
          plugins: {
            legend: {
              display: true,
              position: isPie ? 'bottom' : 'top'
            }
          },
          ...(isPie
            ? {
                circumference: Math.PI * 2,
                rotation: -Math.PI / 2
              }
            : {
                scales: {
                  x: {
                    title: {
                      display: true,
                      text: this.capitalizedTimeInterval
                    }
                  },
                  y: {
                    beginAtZero: true
                  }
                }
              })
        }
      };

      this.chartInstance = new Chart(ctx, config);
      this.$emit('chartDataGenerated', chartData);

    },
    updateChart() {
      if (this.droppedOptions.length) this.renderChart();
    },
getChartData() {
  if (this.savedData) {
    return this.savedData;
  }

  const labels =
    this.timeInterval === 'daywise'
      ? ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']
      : ['00:00', '06:00', '12:00', '18:00', '23:59'];

  const colors =
    this.droppedColors.length > 0
      ? this.droppedColors
      : ['#34d399', '#60a5fa', '#fbbf24', '#f87171', '#a78bfa'];

  // Static dataset per option
  const staticDataMap = {
    'Total Call Initiated': [10, 20, 50, 40, 30],
    'Call Dailed Out': [15, 25, 35, 25, 55],
    'Answered Calls': [12, 42, 32, 42, 52],
    'Unanswered Calls': [8, 48, 18, 38, 28],
    'Failed Calls': [5, 25, 45, 35, 25],
    'Dropped Calls': [7, 17, 27, 17, 47],
    'Busy Calls': [11, 21, 31, 21, 51],
    'No Answers': [6, 36, 26, 36, 46],
    'Invalid Number': [9, 19, 29, 49, 19]
  };

  if (this.chartType === 'pie') {
   
    const data = this.droppedOptions.map(opt => {
      return staticDataMap[opt]?.[0] || 0; 
    });

    return {
      labels: this.droppedOptions,
      datasets: [{
        label: 'Pie Chart',
        data,
        backgroundColor: colors.slice(0, this.droppedOptions.length),
        borderWidth: 1
      }]
    };
  }


  return {
    labels,
    datasets: this.droppedOptions.map((opt, i) => ({
      label: opt,
      data: staticDataMap[opt] || [0, 0, 0, 0, 0],
      backgroundColor: colors[i % colors.length],
      borderColor: colors[i % colors.length],
      borderWidth: 1,
      fill: this.chartType !== 'line'
    }))
  };
}
  }
};
</script>

<style scoped>
.chart-header {
  margin-top: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  flex-wrap: wrap;
}

.chart-heading {
  font-size: 1.1rem;
  font-weight: 600;
  color: #065f46;
  user-select: none;
  margin: 0;
}

.chart-controls {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-top: 8px;
}

.dropdown-group {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.875rem;
}
.dropdown-group select {
  padding: 4px 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.chart-preview-card {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 12px;
  margin-top: 25px;
  background-color: #f0fff4;
  border: 2px solid #cbd5e0;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.app-container.dark .chart-preview-card {
  background-color: #2d3748;
  border-color: #4fd1c5;
}

.chart-preview-card.pie-small {
  justify-content: flex-start;
}

canvas {
  flex: 1;
  width: 100% !important;
  height: 100% !important;
  max-height: 300px;
  margin-top: 12px;
}
</style>
