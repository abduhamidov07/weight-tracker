<template enter-class="container">
  <main class="row row-cols-md-2 d-flex justify-content-center mx-5">
    <div class="card border-0 p-0">
      <h1 class="text-center my-3">Weight Tracker</h1>

      <form class="form d-flex flex-column gap-2" @submit.prevent="addWeight">
        <label for="input" class="fs-5">Your weight</label>
        <input
          type="number"
          id="input"
          v-model="weightInput"
          autocomplete="off"
          placeholder="write your weight"
          class="rounded-3 p-1 fs-5"
        />

        <input
          type="submit"
          value="Add weight"
          class="rounded-3 border-0 py-1 fs-5"
        />
      </form>

      <div v-if="weights && weights.length > 0">
        <h2 class="text-center mt-3">Last 7 day</h2>

        <div class="canvas-box">
          <canvas ref="weightChartEl"></canvas>
        </div>

        <div class="weight-history">
          <h2 class="text-center">Weight history</h2>
          <ul class="p-0">
            <li
              v-for="weight in weights"
              class="d-flex justify-content-between align-items-center border border-secondary p-2 my-2 rounded-4"
            >
              <p class="fs-5 m-0">
                {{ weight.weight }} <span class="text-secondary">kg</span>
              </p>
              <small class="text-dark">
                {{ new Date(weight.date).toLocaleDateString() }}
              </small>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref();

// const currentWeight = computed(() => {
//   return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: '' };
// });

function addWeight() {
  if (weightInput.value !== "") {
    weights.value.unshift({
      weight: weightInput.value,
      date: new Date().getTime(),
    });
    weightInput.value = "";
  }
}

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => weight.weight)
        .slice(-7);

      weightChart.value.update();
      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((weight) => new Date(weight.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: ws
                .sort((a, b) => a.date - b.date)
                .map((weight) => weight.weight),
              backgroundColor: "hsla(220, 60%, 92%, 1)",
              borderColor: "#000",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<style scoped></style>
