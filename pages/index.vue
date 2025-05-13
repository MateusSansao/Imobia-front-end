<template>
  <div class="container-index">
    <v-tabs bg-color="black" class="container-button">
      <v-tab @click="currentView = 'Dashboard'" class="button">Dashboard</v-tab>
      <v-tab @click="currentView = 'List'" class="button">Lista</v-tab>
    </v-tabs>

    <div class="dashboard">
      <component :is="currentViewComponent" />
    </div>
  </div>
</template>

<style scoped>
.container-index {
  display: flex;
  justify-content: center;
  align-items: start;
  width: 100%;
  height: 100vh;
  flex-direction: column;
}

.container-button {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.button {
  display: flex;
  width: 50%;
  justify-content: center;
  align-items: center;
}

.dashboard {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background-color: #1d1d1d;
}
</style>

<script setup>
import { ref, computed } from "vue";
import Dashboard from "/components/dashboard.vue";
import List from "/components/list.vue";

const currentView = ref("Dashboard");

const currentViewComponent = computed(() => {
  return currentView.value === "Dashboard" ? Dashboard : List;
});
</script>
