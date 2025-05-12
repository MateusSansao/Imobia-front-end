<template>
  <div class="dashboard-container">
    <h1 class="dashboard-title">Dashboard de Imóveis</h1>

    <!-- Card com o Total de Imóveis -->
    <div class="stats-card">
      <h2>Total de Imóveis Disponíveis</h2>
      <div class="stats-value">{{ dadosImoveis.total }}</div>
    </div>

    <div class="charts-grid">
      <div class="chart-container">
        <h2>Quantidade de Imóveis por Tipo</h2>
        <client-only>
          <Pie
            v-if="dadosPorTipo && dadosPorTipo.labels && dadosPorTipo.datasets"
            :data="dadosPorTipo"
            :options="opcoesPie"
          />
          <div v-else class="loading">Carregando gráfico...</div>
        </client-only>
      </div>
    </div>
  </div>
</template>

<script>
import { Pie } from "vue-chartjs";
import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement } from "chart.js";

ChartJS.register(Title, Tooltip, Legend, ArcElement);

export default {
  name: "DashboardImoveis",
  components: {
    Pie,
  },
  data() {
    return {
      dadosImoveis: {
        total: 1250,
        porTipo: [
          { tipo: "Apartamento", quantidade: 650 },
          { tipo: "Casa", quantidade: 150 },
          { tipo: "Cobertura", quantidade: 100 },
          { tipo: "Kitnet", quantidade: 80 },
          { tipo: "Terreno", quantidade: 70 },
        ],
      },
      dadosPorTipoInit: {
        labels: [],
        datasets: [
          {
            backgroundColor: [
              "#4c6ef5",
              "#3cba9f",
              "#ffcc00",
              "#ff6384",
              "#9966ff",
            ],
            data: [],
          },
        ],
      },
    };
  },
  computed: {
    dadosPorTipo() {
      if (
        !this.dadosImoveis ||
        !this.dadosImoveis.porTipo ||
        !this.dadosImoveis.porTipo.length
      ) {
        return this.dadosPorTipoInit;
      }

      return {
        labels: this.dadosImoveis.porTipo.map((item) => item.tipo),
        datasets: [
          {
            backgroundColor: [
              "#4c6ef5",
              "#3cba9f",
              "#ffcc00",
              "#ff6384",
              "#9966ff",
            ],
            data: this.dadosImoveis.porTipo.map((item) => item.quantidade),
          },
        ],
      };
    },
    opcoesPie() {
      return {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: "right",
          },
        },
      };
    },
  },
  mounted() {
    console.log("Dashboard de Imóveis montado");
  },
};
</script>

<style scoped>
.dashboard-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.dashboard-title {
  text-align: center;
  margin-bottom: 30px;
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.stats-card {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 30px;
  text-align: center;
}

.stats-card h2 {
  font-size: 18px;
  margin-bottom: 10px;
  color: #555;
}

.stats-value {
  font-size: 36px;
  font-weight: bold;
  color: #4c6ef5;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 20px;
}

.chart-container {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
  height: 350px;
}

.chart-container h2 {
  font-size: 16px;
  margin-bottom: 15px;
  color: #555;
  text-align: center;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 80%;
  color: #888;
  font-style: italic;
}
</style>
