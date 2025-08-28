<template>
  <div>
    <div class="filters">
      <label
        v-for="item in sentimentData"
        :key="item._id"
        class="filter-chip"
        :class="{ 'selected': selectedIds.includes(item._id) }"
      >
        <input
          type="checkbox"
          v-model="selectedIds"
          :value="item._id"
          class="hidden-checkbox"
        />
        {{ item._id }}
      </label>
    </div>

    <div ref="container" class="container">
      <div
        v-for="chart in charts"
        :key="chart._id"
        class="circle"
        v-tooltip="{
          content: getTooltip(chart),
          html: true,
          placement: 'top',
        }"
        :style="{
          width: chart.r * 2 + 'px',
          height: chart.r * 2 + 'px',
          left: chart.x + 'px',
          top: chart.y + 'px',
          transform: 'translate(-50%, -50%)',
        }"
      >
        <VChart
          :option="getChartOption(chart)"
          autoresize
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import VChart from "vue-echarts";
import { use } from "echarts/core";
import { PieChart } from "echarts/charts";
import { TooltipComponent } from "echarts/components";
import { SVGRenderer } from "echarts/renderers";
import { sentimentData } from "@/data/sentiments";

use([PieChart, TooltipComponent, SVGRenderer]);

const container = ref(null);
const charts = ref([]);
const COLORS = ["#4caf50", "#ffc107", "#f44336"];

const selectedIds = ref(sentimentData.value.map(item => item._id));

const filteredData = computed(() =>
  sentimentData.value.filter(item => selectedIds.value.includes(item._id))
);

function getRadius(total, baseRadius, maxTotal) {
  return baseRadius + (total / maxTotal) * baseRadius * 2; 
}

function calculateCharts(data, width, height) {
  if (!data.length) {
    charts.value = [];
    return;
  }

  const baseRadius = Math.min(width, height) / 25;
  const maxTotal = Math.max(...data.map(d => d.total));

  const maxItem = data.reduce((a, b) => (a.total > b.total ? a : b));
  const others = data.filter((d) => d !== maxItem);

  const centerR = getRadius(maxItem.total, baseRadius, maxTotal);
  const centerX = width / 2;
  const centerY = height / 2;

  const centerChart = {
    ...maxItem,
    r: centerR,
    x: centerX,
    y: centerY,
  };

  const maxOtherR = Math.max(...others.map((o) => getRadius(o.total, baseRadius, maxTotal)));
  const padding = baseRadius * 2;
  const orbitRadius = centerR + maxOtherR + padding;

  const angleStep = (2 * Math.PI) / others.length;

  const otherCharts = others.map((d, i) => {
    const r = getRadius(d.total, baseRadius, maxTotal);
    const angle = i * angleStep;

    return {
      ...d,
      r,
      x: centerX + orbitRadius * Math.cos(angle),
      y: centerY + orbitRadius * Math.sin(angle),
    };
  });

  charts.value = [centerChart, ...otherCharts];
}

function getChartOption(chart) {
  return {
    series: [
      {
        name: chart._id,
        type: "pie",
        radius: ["70%", "90%"],
        label: {
          show: true,
          position: "center",
          formatter: chart._id,
          color: "#333",
          fontSize: 12,
          fontWeight: "bold",
        },
        emphasis: { scale: false },
        data: [
          { value: chart.positivo ?? 0, name: "Positivo", itemStyle: { color: COLORS[0] } },
          { value: chart.neutro ?? 0, name: "Neutro", itemStyle: { color: COLORS[1] } },
          { value: chart.negativo ?? 0, name: "Negativo", itemStyle: { color: COLORS[2] } },
        ],
      },
    ],
  };
}

function getTooltip(chart) {
  return `
    <div class="custom-tooltip">
      <strong>${chart._id}</strong>
      <div>Positivo: ${chart.positivo}</div>
      <div>Neutro: ${chart.neutro}</div>
      <div>Negativo: ${chart.negativo}</div>
    </div>
  `;
}

function updateLayout() {
  if (!container.value) return;
  const width = container.value.clientWidth;
  const height = container.value.clientHeight;
  calculateCharts(filteredData.value, width, height);
}

watch(filteredData, updateLayout, { immediate: true });

onMounted(() => {
  updateLayout();
  window.addEventListener("resize", updateLayout);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", updateLayout);
});
</script>

<style scoped>
.container {
  position: relative;
  width: 100%;
  height: 98vh;
  overflow: hidden;
}

.circle {
  position: absolute;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 0 10px rgb(58 142 230 / 0.3);
  overflow: hidden;
  user-select: none;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: left 0.5s ease, top 0.5s ease, width 0.5s ease, height 0.5s ease;
}

.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  padding: 1rem;
}

.filter-chip {
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  border: 2px solid #ddd;
  border-radius: 9999px;
  background-color: #f8f8f8;
  color: #333;
  cursor: pointer;
  transition: all 0.2s ease;
  user-select: none;
}

.filter-chip:hover {
  border-color: #aaa;
}

.filter-chip.selected {
  background-color: #007bff;
  color: white;
  border-color: #007bff;
}

.hidden-checkbox {
  display: none;
}
</style>