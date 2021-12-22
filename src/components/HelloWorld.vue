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

        <!-- Navbar dropdowns -->
        <!--
        <b-nav-item-dropdown text="Lang" right>
          <b-dropdown-item href="#">EN</b-dropdown-item>
          <b-dropdown-item href="#">ES</b-dropdown-item>
          <b-dropdown-item href="#">RU</b-dropdown-item>
          <b-dropdown-item href="#">FA</b-dropdown-item>
        </b-nav-item-dropdown>
        -->

        <b-nav-item-dropdown text="User" right>
          <b-dropdown-item href="#">Account</b-dropdown-item>
          <b-dropdown-item href="#">Settings</b-dropdown-item>
        </b-nav-item-dropdown>
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

  </div>
</template>

<script>

import PaginaArtisti from "@/components/PaginaArtisti";
import Gruppi from "@/components/Gruppi";
import Città from "@/components/Città";
import Concerti from "@/components/Concerti";
import Scalette from "@/components/Scalette";
import Canzoni from "@/components/Canzoni";

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
    Canzoni
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

    console.log(this.page);


    /*
    var query = "select ?artista where { ?artista rdf:type meo:Artista } limit 100";

    const myFetcher = new SparqlEndpointFetcher({
      method: 'GET',                   // A custom HTTP method for issuing (non-update) queries, defaults to POST. Update queries are always issued via POST.
      fetch: fetch,                     // A custom fetch-API-supporting function
      prefixVariableQuestionMark: false // If variable names in bindings should be prefixed with '?', defaults to false
    });

    const tripleStream = await myFetcher.fetchTriples('http://localhost:7200/sparql', query);
    tripleStream.on('data', (triple) => console.log(triple));

     */

    /*
    const SparqlClient = require('sparql-http-client')

    const endpointUrl = 'http://localhost:7200/repositories/MusicalEventsOntology'
    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select ?artista
      where {
      ?artista rdf:type meo:Artista
      } limit 100
    `;

    const client = new SparqlClient({endpointUrl})
    const stream = await client.query.select(query)

    stream.on('data', row => {
      Object.entries(row).forEach(([key, value]) => {
        console.log(`${key}: ${value.value} (${value.termType})`)
      })
    })

    stream.on('error', err => {
      console.error(err)
    })

     */
  },

  methods: {
    changePage(page) {
      this.page = page;
      console.log(this.page);
    },
    foo: function() {
      alert('this is A.foo');
    },
    async makeQuery(query) {
      var queryData = [];

      const client = new SparqlClient({endpointUrl})
      const stream = await client.query.select(query)

      var i = 0;
      var nrow = 0;

      /*Recupero il numero di righe della nostra risposta*/
      stream.on('data', () => {
        nrow++;
      })

      /*Inizio l'oggetto da passare al component BTable*/
      queryData = new Array(Object.entries(nrow).length);

      stream.on('data', row => {
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
