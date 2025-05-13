<script setup>
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import {
  faHome,
  faDollarSign,
  faBookmark,
  faCity,
} from "@fortawesome/free-solid-svg-icons";
import { library } from "@fortawesome/fontawesome-svg-core";
import { Pie } from "vue-chartjs";
import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement } from "chart.js";
import { ref, onMounted } from "vue";

library.add(faHome, faDollarSign, faBookmark, faCity);

ChartJS.register(Title, Tooltip, Legend, ArcElement);

const dadosImoveis = ref({
  porTipo: [
    { tipo: "Apartamento", quantidade: 0 },
    { tipo: "Casa", quantidade: 0 },
    { tipo: "Cobertura", quantidade: 0 },
    { tipo: "Kitnet", quantidade: 0 },
    { tipo: "Terreno", quantidade: 0 },
  ],
});

// Definição de dados iniciais para o gráfico
const dadosPorTipoInit = {
  labels: [],
  datasets: [
    {
      backgroundColor: ["#4c6ef5", "#3cba9f", "#ffcc00", "#ff6384", "#9966ff"],
      data: [],
    },
  ],
};

// Computed properties
const dadosPorTipo = computed(() => {
  if (!dadosImoveis || !dadosImoveis.porTipo || !dadosImoveis.porTipo.length) {
    return dadosPorTipoInit;
  }

  return {
    labels: dadosImoveis.porTipo.map((item) => item.tipo),
    datasets: [
      {
        backgroundColor: [
          "#4c6ef5",
          "#3cba9f",
          "#ffcc00",
          "#ff6384",
          "#9966ff",
        ],
        data: dadosImoveis.porTipo.map((item) => item.quantidade),
      },
    ],
  };
});

const opcoesPie = computed(() => {
  return {
    responsive: true,
    maintainAspectRatio: true,
    plugins: {
      legend: {
        position: "right",
      },
    },
  };
});

const full = ref(0);
const fullAvailable = ref(0);
const valueAverage = ref(0);
const cidadeMaisBarata = ref("");
const cidadeMaisImoveis = ref("");

const { data } = await useFetch(
  "https://apicontrole.useimobia.com.br/api/desafio-imobia/imoveis"
);

onMounted(async () => {
  console.log("dashboard");

  if (data.value) {
    full.value = data.value.length;
  }

  if (data.value) {
    fullAvailable.value = data.value.filter(
      (imovel) => imovel.situacao === "disponivel"
    ).length;
  }

  if (data.value) {
    const alugueis = data.value.filter((imovel) => imovel.valor_aluguel);
    const soma = alugueis.reduce(
      (total, imovel) => total + Number(imovel.valor_aluguel),
      0
    );

    valueAverage.value = alugueis.length > 0 ? soma / alugueis.length : 0;
  }

  if (data.value) {
    const alugueis = data.value.filter((imovel) => imovel.valor_aluguel);

    if (alugueis.length > 0) {
      const imovelMaisBarato = alugueis.reduce((maisBarato, atual) => {
        return Number(atual.valor_aluguel) < Number(maisBarato.valor_aluguel)
          ? atual
          : maisBarato;
      });

      cidadeMaisBarata.value = imovelMaisBarato.cidade;
    }
  }

  if (data.value) {
    const counts = {};

    for (const imovel of data.value) {
      counts[imovel.cidade] = (counts[imovel.cidade] || 0) + 1;
    }

    cidadeMaisImoveis.value = Object.entries(counts).reduce((a, b) =>
      a[1] > b[1] ? a : b
    )[0];
  }
});
</script>

<template>
  <div class="container">
    <div class="row-graphic">
      <div class="session-02">
        <div class="container-cards">
          <div class="card">
            <div class="conteiner-icon-home">
              <font-awesome-icon :icon="['fas', 'house']" class="icon-home" />
            </div>
            <div>
              <h1 class="info-card-h1">Total de imóveis</h1>
              <p class="info-card-p">{{ full }}</p>
            </div>
          </div>
          <div class="card">
            <div class="conteiner-icon-home-available">
              <font-awesome-icon
                :icon="['fas', 'house']"
                class="icon-home-available"
              />
            </div>
            <div>
              <h1 class="info-card-h1">Total de imóveis Disponiveis</h1>
              <p class="info-card-p">{{ fullAvailable }}</p>
            </div>
          </div>
          <div class="card">
            <div class="conteiner-icon-money">
              <font-awesome-icon
                :icon="['fas', 'dollar-sign']"
                class="icon-money"
              />
            </div>
            <div>
              <h1 class="info-card-h1">Preço médio</h1>
              <p class="info-card-p">R${{ valueAverage.toFixed(2) }}</p>
            </div>
          </div>
        </div>
        <div class="container-cards">
          <div class="card">
            <div class="conteiner-icon-type">
              <font-awesome-icon
                :icon="['fas', 'bookmark']"
                class="icon-type"
              />
            </div>
            <div>
              <h1 class="info-card-h1">Cidade com menor preço</h1>
              <p class="info-card-p">{{ cidadeMaisBarata }}</p>
            </div>
          </div>
          <div class="card">
            <div class="conteiner-icon-city">
              <font-awesome-icon :icon="['fas', 'city']" class="icon-city" />
            </div>
            <div>
              <h1 class="info-card-h1">Cidade com mais imóvel</h1>
              <p class="info-card-p">{{ cidadeMaisImoveis }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="charts-grid">
        <div class="chart-container">
          <h2>Quantidade de Imóveis por Tipo</h2>
          <client-only>
            <Pie
              v-if="
                dadosPorTipo && dadosPorTipo.labels && dadosPorTipo.datasets
              "
              :data="dadosPorTipo"
              :options="opcoesPie"
            />
            <div v-else class="loading">Carregando gráfico...</div>
          </client-only>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  padding: 16px;
  gap: 30px;
}

.session-filter {
  display: flex;
  justify-content: start;
  align-items: center;
  width: 100%;
  height: 10vh;
  padding-left: 10px;
  gap: 20px;
}

.card-filter {
  display: flex;
  justify-content: flex-center;
  align-items: start;
  flex-direction: column;
  width: 20%;
}

.row-graphic {
  display: flex;
  justify-content: center;
  align-items: start;
  flex-direction: row;
  width: 100%;
}

.input-filter {
  width: 100%;
  height: 40px;
  background-color: white;
  border-radius: 8px;
}

.label-filter {
  color: white;
}

.session {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;
}

.session-02 {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: auto;
  gap: 30px;
  padding: 10px;
}

.container-cards {
  display: flex;
  justify-content: start;
  align-items: center;
  flex-direction: column;
  width: 100%;
  gap: 30px;
}

.card {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  width: 100%;
  border-radius: 10px;
  background-color: white;
  padding: 10px;
  gap: 10px;
}

.info-card-h1 {
  font-size: 20px;
  font-weight: 400;
  color: gray;
}

.info-card-p {
  font-size: 18px;
  font-weight: 800;
}

.icon-home {
  width: 24px;
  height: auto;
  opacity: 60%;
  color: rgb(255, 0, 55);
}

.conteiner-icon-home {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  background: rgba(255, 0, 13, 0.356);
}

.icon-home-available {
  width: 24px;
  height: auto;
  opacity: 60%;
  color: blue;
}

.conteiner-icon-home-available {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  background: rgba(0, 102, 255, 0.356);
}

.icon-money {
  width: 18px;
  height: auto;
  opacity: 60%;
  color: green;
}

.conteiner-icon-money {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  background: rgba(0, 255, 179, 0.356);
}

.icon-type {
  width: 18px;
  height: auto;
  opacity: 60%;
  color: rgb(255, 115, 0);
}

.conteiner-icon-type {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  background: rgba(229, 255, 0, 0.356);
}

.icon-city {
  width: 24px;
  height: auto;
  opacity: 60%;
  color: rgb(0, 119, 255);
}

.conteiner-icon-city {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 60px;
  height: 60px;
  border-radius: 30px;
  background: rgba(0, 183, 255, 0.356);
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 20px;
  width: 760px;
  height: auto;
  padding: 10px;
}

.chart-container {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
}
</style>
