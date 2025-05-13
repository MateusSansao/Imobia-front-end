<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { useFetch } from "#app";
import _ from "lodash";

const imoveis = ref([]);
const quantidadeExibir = ref(10);
const estadoSelecionado = ref(null);
const cidadeSelecionada = ref(null);
const situacaoSelecionada = ref(null);
const loading = ref(false);

const estados = ref([]);
const cidadesMap = ref({});
const situacoes = ["Disponível", "Locado", "Indisponível"];

function normalize(str) {
  return (str || "")
    .toString()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .trim()
    .toLowerCase();
}

const fetchImoveis = async () => {
  loading.value = true;
  try {
    const { data } = await useFetch(
      "https://apicontrole.useimobia.com.br/api/desafio-imobia/imoveis"
    );

    if (data.value && Array.isArray(data.value)) {
      imoveis.value = data.value.map((item) => ({
        ...item,
        show: false,
      }));

      estados.value = _.sortBy(
        _.uniq(
          data.value
            .map((i) => i.estado)
            .filter(Boolean)
            .map((e) => e.trim())
        )
      );

      cidadesMap.value = _.chain(data.value)
        .groupBy("estado")
        .mapValues((imoveisDoEstado) =>
          _.chain(imoveisDoEstado)
            .map((i) => i.cidade)
            .filter(Boolean)
            .map((c) => c.trim())
            .uniq()
            .sortBy()
            .value()
        )
        .value();
    }
  } catch (error) {
    console.error("Erro ao buscar imóveis:", error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchImoveis();
});

const cidadesDisponiveis = computed(() => {
  if (!estadoSelecionado.value) return [];

  return cidadesMap.value[estadoSelecionado.value] || [];
});

watch(estadoSelecionado, () => {
  cidadeSelecionada.value = null;
});

const imoveisFiltrados = computed(() => {
  return _.chain(imoveis.value)
    .filter((imovel) => {
      if (
        estadoSelecionado.value &&
        normalize(imovel.estado) !== normalize(estadoSelecionado.value)
      ) {
        return false;
      }

      if (
        cidadeSelecionada.value &&
        normalize(imovel.cidade) !== normalize(cidadeSelecionada.value)
      ) {
        return false;
      }

      if (
        situacaoSelecionada.value &&
        normalize(imovel.situacao) !== normalize(situacaoSelecionada.value)
      ) {
        return false;
      }

      return true;
    })
    .take(quantidadeExibir.value)
    .value();
});
</script>

<template>
  <v-container style="max-width: 100%; margin: auto">
    <div class="container-filter">
      <v-row class="mb-4" justify="start">
        <v-col cols="12" sm="3" md="2">
          <v-select
            v-model="quantidadeExibir"
            :items="[5, 10, 20, 50, 100, 500]"
            label="Qtd. imóveis"
            class="bg-white"
            outlined
            dense
            hide-details
          />
        </v-col>

        <v-col cols="12" sm="3" md="2">
          <v-select
            v-model="estadoSelecionado"
            :items="estados"
            label="Estado"
            class="bg-white"
            outlined
            dense
            hide-details
            clearable
            :loading="loading"
          />
        </v-col>

        <v-col cols="12" sm="3" md="2">
          <v-select
            v-model="cidadeSelecionada"
            :items="cidadesDisponiveis"
            label="Cidade"
            class="bg-white"
            outlined
            dense
            hide-details
            :disabled="!estadoSelecionado"
            clearable
            :loading="loading && estadoSelecionado"
            item-title="text"
            item-value="value"
            return-object
          />
        </v-col>

        <v-col cols="12" sm="3" md="2">
          <v-select
            v-model="situacaoSelecionada"
            :items="situacoes"
            label="Situação"
            class="bg-white"
            outlined
            dense
            hide-details
            clearable
          />
        </v-col>
      </v-row>
    </div>

    <v-fade-transition>
      <div v-if="loading" class="d-flex justify-center my-5">
        <v-progress-circular indeterminate color="primary" />
      </div>

      <div
        v-else
        style="max-height: 70vh; overflow-y: auto"
        class="scroll-wrapper"
      >
        <v-row dense>
          <v-col
            v-for="imovel in imoveisFiltrados"
            :key="imovel.id"
            cols="12"
            sm="6"
            md="4"
          >
            <v-card>
              <v-img height="120px" src="/images/fundo.svg" cover />

              <v-card-title>{{ imovel.nome || "Sem nome" }}</v-card-title>
              <v-card-subtitle>
                {{ imovel.cidade || "Sem cidade" }} -
                {{ imovel.estado || "Sem estado" }}
              </v-card-subtitle>

              <v-card-actions>
                <v-chip
                  :color="getSituacaoColor(imovel.situacao)"
                  text-color="white"
                  small
                >
                  {{ imovel.situacao || "Sem situação" }}
                </v-chip>
                <v-spacer></v-spacer>
                <v-btn
                  :icon="imovel.show ? 'mdi-chevron-up' : 'mdi-chevron-down'"
                  @click="imovel.show = !imovel.show"
                />
              </v-card-actions>

              <v-expand-transition>
                <div v-show="imovel.show">
                  <v-divider />
                  <v-card-text>
                    <strong>Bairro:</strong> {{ imovel.bairro || "-" }}<br />
                    <strong>Rua:</strong> {{ imovel.rua || "-" }},
                    {{ imovel.numero || "-" }}
                    <br />
                    <strong>Aluguel:</strong> R$
                    {{
                      imovel.valor_aluguel
                        ? imovel.valor_aluguel.toFixed(2)
                        : "-"
                    }}<br />
                    <strong>IPTU:</strong
                    >{{
                      imovel.valor_iptu ? imovel.valor_iptu.toFixed(2) : "-"
                    }}
                  </v-card-text>
                </div>
              </v-expand-transition>
            </v-card>
          </v-col>
          <v-col v-if="imoveisFiltrados.length === 0" cols="12">
            <v-alert type="info" text> Nenhum imóvel encontrado. </v-alert>
          </v-col>
        </v-row>
      </div>
    </v-fade-transition>
  </v-container>
</template>

<script>
export default {
  methods: {
    getSituacaoColor(situacao) {
      if (!situacao) return "grey";

      const normalizado = situacao.toLowerCase().trim();

      switch (normalizado) {
        case "disponível":
          return "success";
        case "locado":
          return "error";
        case "indisponível":
          return "warning";
        default:
          return "primary";
      }
    },
  },
};
</script>

<style scoped>
.scroll-wrapper {
  scrollbar-width: none;
}

.scroll-wrapper::-webkit-scrollbar {
  display: none;
}

.container-filter {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
  margin-bottom: 16px;
}

.v-select {
  margin-bottom: 0;
}

@media (max-width: 600px) {
  .container-filter {
    padding: 10px;
  }
}
</style>
