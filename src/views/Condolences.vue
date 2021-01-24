<template>
  <div class="container px-2 mb-20">
    <Jumbotron>
      <div class="search-wrapper flex items-baseline flex-col sm:flex-row">
        <h3 class="text-2xl text-gray-900 font-semibold">Condolências</h3>
        <div class="sm:ml-auto">
          <input
            class="inline border rounded-sm px-2 outline-none"
            type="text"
            v-model="search"
            placeholder="Buscar por nome.."
          />
          <button class="inline p-3" @click="searchEvent()">
            <img
              class="w-4"
              src="../assets/images/icon-search.svg"
              alt="Ícone de busca"
            />
          </button>
        </div>
      </div>
      <hr />

      <div class="mb-4 mt-5 sm:grid sm:grid-cols-2 lg:grid-cols-3 sm:gap-4">
        <p v-if="condolences.length <= 0" class="text-xl text-gray-900">
          Nenhum registro encontrado
        </p>
        <div
          v-else
          v-for="condolence in condolences"
          v-bind:key="condolence.id"
        >
          <CardCondolence :condolencia="condolence"></CardCondolence>
        </div>
      </div>
      <div
        v-show="condolences.length > 0"
        class="flex flex-col items-center space-y-3"
      >
        <paginate
          v-model="page"
          :page-count="pageCount"
          :page-range="3"
          :margin-pages="2"
          :click-handler="clickCallback"
          :prev-text="'Voltar'"
          :next-text="'Próxima'"
          :container-class="'pagination'"
          :page-class="'page-item'"
        >
        </paginate>
      </div>
    </Jumbotron>
    <loading
      :active.sync="isLoading"
      :can-cancel="false"
      :is-full-page="fullPage"
    >
    </loading>
  </div>
</template>

<script>
import axios from "axios";
import CardCondolence from "@/components/CardCondolence";
import Jumbotron from "@/components/Jumbotron";

// Import component
import Loading from "vue-loading-overlay";
// Import stylesheet
import "vue-loading-overlay/dist/vue-loading.css";

export default {
  props: {},
  name: "condolences",
  components: {
    CardCondolence,
    Jumbotron,
    Loading,
  },
  data() {
    return {
      condolences: [],
      pessoa: null,
      showModal: false,
      isLoading: false,
      fullPage: true,
      search: "",
      isSearch: false,
      page: 1,
      itemsPage: 12,
      pageCount: 1,
    };
  },
  mounted() {
    this.isLoading = true;
    this.getCondolencias(
      `http://api.memorialavarc.com.br/api/Mensagems/status?status=Aprovado&qtd_registros=${this.itemsPage}&pagina=1`
    );
  },
  methods: {
    searchEvent() {
      this.isSearch = true;
      let name = this.search.trim().split(" ")[0];
      if (name === "") {
        this.search = "";
        this.getCondolencias(
          `http://api.memorialavarc.com.br/api/Mensagems/status?status=Aprovado&qtd_registros=${this.itemsPage}&pagina=1`
        );
      } else {
        this.getCondolencias(
          `http://api.memorialavarc.com.br/api/Mensagems/Nome?nome_vitima=${name}&qtd_registros=${this.itemsPage}&pagina=1`
        );
      }
    },

    clickCallback: function(pageNum) {
      let route = "";
      if (this.isSearch) {
        route = `http://api.memorialavarc.com.br/api/Mensagems/Nome?nome_vitima=${name}&qtd_registros=${this.itemsPage}&pagina=${pageNum}`;
      } else {
        route = `http://api.memorialavarc.com.br/api/Mensagems/status?status=Aprovado&qtd_registros=${this.itemsPage}&pagina=${pageNum}`;
      }
      this.getCondolencias(route);
    },

    getCondolencias: function(route) {
      this.isLoading = true;
      axios
        .get(route)
        .then((response) => {
          this.condolences = response.data;
          this.pageCount = response.data[0].qtd_paginas;
        })
        .catch((err) => {
          console.log(new Error(err.message));
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
  },
};
</script>
