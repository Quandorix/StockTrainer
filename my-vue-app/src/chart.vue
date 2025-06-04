```vue
<template>
  <div class="chart-container">
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
import Chart from 'chart.js/auto';

export default {
  name: 'Chart',
  props: {
    currentStock: {
      type: String,
      required: true
    },
    stockPrices: {
      type: Object,
      required: true
    }
  },
  setup(props) {
    const chartCanvas = ref(null);
    let chartInstance = null;

    function updateChart() {
      if (!props.currentStock || !chartCanvas.value || !props.stockPrices[props.currentStock]) return;

      const symbol = props.currentStock;
      const ctx = chartCanvas.value.getContext('2d');

      if (chartInstance) {
        chartInstance.destroy();
      }

      chartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: props.stockPrices[symbol]?.history?.map(point =>
            new Date(point.date).toLocaleDateString('de-DE', { month: 'short', day: 'numeric' })
          ) || [],
          datasets: [{
            label: `${symbol} Preis ($)`,
            data: props.stockPrices[symbol]?.history?.map(point => point.price) || [],
            borderColor: '#3B82F6',
            backgroundColor: 'rgba(59, 130, 246, 0.1)',
            borderWidth: 3,
            fill: true,
            tension: 0.4,
            pointBackgroundColor: '#3B82F6',
            pointBorderColor: '#ffffff',
            pointBorderWidth: 2,
            pointRadius: 4,
            pointHoverRadius: 6
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: {
              display: true,
              text: `${symbol} - 30 Tage Chart`,
              font: {
                size: 16,
                weight: 'bold'
              },
              color: '#374151'
            },
            legend: {
              display: true,
              labels: {
                font: {
                  size: 12
                },
                color: '#6B7280'
              }
            }
          },
          scales: {
            y: {
              beginAtZero: false,
              title: {
                display: true,
                text: 'Preis ($)',
                font: {
                  weight: 'bold'
                },
                color: '#374151'
              },
              grid: {
                color: 'rgba(0, 0, 0, 0.1)'
              },
              ticks: {
                color: '#6B7280',
                callback: function(value) {
                  return '$' + value.toFixed(2);
                }
              }
            },
            x: {
              title: {
                display: true,
                text: 'Datum',
                font: {
                  weight: 'bold'
                },
                color: '#374151'
              },
              grid: {
                color: 'rgba(0, 0, 0, 0.1)'
              },
              ticks: {
                color: '#6B7280'
              }
            }
          },
          interaction: {
            intersect: false,
            mode: 'index'
          }
        }
      });
    }

    watch([() => props.currentStock, () => props.stockPrices], () => {
      updateChart();
    }, { deep: true });

    return { chartCanvas };
  }
}
</script>

<style scoped>
.chart-container {
  background: white;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
  height: 400px;
}
</style>
```