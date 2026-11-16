<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import Chart from 'chart.js/auto'
import { formatNumber } from '../utils/formatters'

const props = defineProps({ currencies: { type: Array, default: () => [] }, compact: Boolean })
const canvas = ref(null)
let chart
const ranked = computed(() => props.currencies.slice().sort((a, b) => b.valueMzn - a.valueMzn).slice(0, props.compact ? 6 : 12))

function renderChart() {
  if (!canvas.value) return
  chart?.destroy()
  chart = new Chart(canvas.value, {
    type: 'bar',
    data: {
      labels: ranked.value.map((item) => item.code),
      datasets: [{
        data: ranked.value.map((item) => item.valueMzn),
        backgroundColor: ranked.value.map((_, index) => index === 0 ? '#d9ab39' : 'rgba(23, 104, 76, .78)'),
        borderRadius: 7,
        borderSkipped: false,
        barThickness: 25,
      }],
    },
    options: {
      indexAxis: 'y',
      responsive: true,
      maintainAspectRatio: false,
      animation: { duration: 450 },
      plugins: {
        legend: { display: false },
        tooltip: { callbacks: { label: (context) => ` ${formatNumber(context.parsed.x)} MZN · exemplo` } },
      },
      scales: {
        x: { beginAtZero: true, grid: { color: '#e9e7e0', drawBorder: false }, border: { display: false }, ticks: { color: '#777d76', font: { family: 'DM Sans' } } },
        y: { grid: { display: false }, border: { display: false }, ticks: { color: '#454b44', font: { family: 'DM Sans', weight: 600 } } },
      },
    },
  })
}

onMounted(renderChart)
watch(ranked, renderChart, { deep: true })
onBeforeUnmount(() => chart?.destroy())
</script>

<template>
  <section class="chart-panel">
    <div class="panel-heading chart-heading">
      <div><span class="section-kicker">DADOS SINTÉTICOS</span><h2>Comparação de valores</h2><p>Valores de exemplo por unidade de moeda, em MZN.</p></div>
      <span class="chart-legend"><i></i> Valor de exemplo</span>
    </div>
    <div v-if="!ranked.length" class="chart-empty"><i class="bi bi-bar-chart"></i><span>Sem valores disponíveis para comparar.</span></div>
    <div v-else class="chart-canvas-wrap" :class="{ 'compact-chart': compact }"><canvas ref="canvas" aria-label="Gráfico de comparação dos valores de demonstração" role="img"></canvas></div>
    <div class="chart-disclaimer"><i class="bi bi-info-circle"></i> Dados sintéticos apenas para demonstração; não representam taxas reais nem histórico de mercado.</div>
  </section>
</template>
