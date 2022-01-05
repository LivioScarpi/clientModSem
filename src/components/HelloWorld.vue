<template>
  <div>
    <b-navbar type="dark" variant="dark">
      <b-navbar-nav>
        <b-nav-item @click="changePage(0)">Home</b-nav-item>
        <b-nav-item @click="changePage(1)">Eventi Live Artisti</b-nav-item>
        <b-nav-item @click="changePage(2)">Gruppi</b-nav-item>
        <b-nav-item @click="changePage(3)">Città</b-nav-item>
        <b-nav-item @click="changePage(4)">Concerti</b-nav-item>
        <b-nav-item @click="changePage(5)">Scalette</b-nav-item>
        <b-nav-item @click="changePage(6)">Canzoni</b-nav-item>
        <b-nav-item @click="changePage(7)">Performance Canzoni</b-nav-item>
        <b-nav-item @click="changePage(8)">Strutture Musicali</b-nav-item>
      </b-navbar-nav>
    </b-navbar>

    <div v-if="this.page === 0">
      Homepage
    </div>
    <div v-else-if="this.page === 1">
      <PaginaArtisti/>
    </div>
    <div v-else-if="this.page === 2">
      <Gruppi/>
    </div>
    <div v-else-if="this.page === 3">
      <Città/>
    </div>
    <div v-else-if="this.page === 4">
      <Concerti/>
    </div>
    <div v-else-if="this.page === 5">
      <Scalette/>
    </div>
    <div v-else-if="this.page === 6">
      <Canzoni/>
    </div>
    <div v-else-if="this.page === 7">
      <PerformanceCanzoni/>
    </div>
    <div v-else-if="this.page === 8">
      <StruttureMusicali/>
    </div>
  </div>
</template>

<script>

import PaginaArtisti from "@/components/PaginaArtisti";
import Gruppi from "@/components/Gruppi";
import Città from "@/components/Città";
import Concerti from "@/components/Concerti";
import Scalette from "@/components/Scalette";
import Canzoni from "@/components/Canzoni";
import PerformanceCanzoni from "@/components/PerformanceCanzoni";
import StruttureMusicali from "./StruttureMusicali";

const SparqlClient = require('sparql-http-client')
const endpointUrl = 'http://localhost:7200/repositories/MEO'

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },

  components: {
    PaginaArtisti,
    Gruppi,
    Città,
    Concerti,
    Scalette,
    Canzoni,
    PerformanceCanzoni,
    StruttureMusicali
  },

  data: function () {
    return {
      page: 0,
      queryData: []
    }
  },

  created() {
    this.$root.$refs.HelloWorld = this;
  },

  async mounted() {
  },

  methods: {
    changePage(page) {
      this.page = page;
      console.log(this.page);
    },
    foo: function() {
      alert('this is A.foo');
    },

    /*
    * Metodo che si occupa di contattare l'endpoint indicato da endpointUrl, di elaborare la risposta ottenuta
    * e riorganizzarla in una struttura adatta per essere utilizzata con le b-table di bootstrap
    */
    async makeQuery(query) {
      var queryData = [];

      const client = new SparqlClient({endpointUrl})
      const stream = await client.query.select(query)

      var i = 0;
      var nrow = 0;

      /* Recupero il numero di righe della nostra risposta */
      stream.on('data', () => {
        nrow++;
      })

      /* Inizializzo l'oggetto da passare al component BTable */
      queryData = new Array(Object.entries(nrow).length);

      stream.on('data', row => {
        console.log(row);

        queryData.push({});

        Object.entries(row).forEach(([key, value]) => {
          queryData[i][key] = value.value.replace('http://www.modsem.org/musicalEventsOntology#', '');
        })

        i++;
      })

      stream.on('error', err => {
        console.error(err)
      })

      return queryData;
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
