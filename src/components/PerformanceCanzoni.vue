<template>
  <div class="m-3">
    <div v-if="artistsLoaded">
      <h3>Performance Canzoni</h3>

      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allEvents" :key="index" @click="callQueryMethods(elem.value)" button>{{ elem.value }}</b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
            <h4>Eventi in cui la performance era in scaletta</h4>
            <div v-if="events.length > 0">
                <b-table striped hover :items="events"></b-table>
              </div>
              <div v-else>
                <h5>La performance non è stata suonata in nessun evento</h5>
              </div>
            <h4>Strumenti con cui la performance è stata suonata</h4>
            <h4>NB! Se nella colonna "Strumento", se presente, non è presente nulla, vuol dire che l'artista ha cantato quella canzone</h4>
            <div v-if="instruments.length > 0">
              <b-table striped hover :items="instruments"></b-table>
            </div>
            <div v-else>
              <h5>La performance non è stata suonata con degli strumenti</h5>
            </div>
          </b-col>
        </b-row>
      </b-container>
    </div>
  </div>
</template>

<script>
import {BContainer, BRow, BCol} from "bootstrap-vue";

const SparqlClient = require('sparql-http-client')
const endpointUrl = 'http://localhost:7200/repositories/MEO'

export default {
  name: "PerformanceCanzoni",

  components: {
    //BButton,
    BContainer, BRow, BCol
  },

  data: function () {
    return {
      eventSelected: null,
      allEvents: [],
      artistsLoaded: false,
      allLoaded: false,
      events: [],
      instruments: [],
      status: 'not_selected',
    }
  },

  async mounted() {

    /*
    * Query per ottenere ttte le performance
    */
    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select ?performanceCanzone
      where {
          ?performanceCanzone rdf:type meo:PerformanceCanzone;
      } limit 100
    `;

    const client = new SparqlClient({endpointUrl})
    const stream = await client.query.select(query)

    var self = this;

    stream.on('data', row => {
      console.log(row);
      Object.entries(row).forEach(([key, value]) => {

        var artistName = value.value.replace('http://www.modsem.org/musicalEventsOntology#Performance', '');

        self.allEvents.push({value: artistName, text: artistName});

        console.log(`${key}: ${value.value} (${value.termType})`)
      })

      self.artistsLoaded = true;
    })

    stream.on('error', err => {
      console.error(err)
    })
  },

  methods: {
    callQueryMethods(songName) {
      this.findInstruments(songName);
      this.findLiveEvents(songName);
    },
    async findLiveEvents(songName) {
      this.events = [];

      var query = "";

      /*
      * Query per ottenere tutti gli eventi in cui quella performance è stata performata
      */
        query = `
                    PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
          PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
          PREFIX owl: <http://www.w3.org/2002/07/owl#>
          PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
          PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
          PREFIX owp: <http://www.ontologydesignpatterns.org/cp/owl/bag.owl#>


          select ?nomeEvento
          where {
              ?evento  rdf:type meo:EventoMusicale;
                       meo:nomeEventoMusicale ?nomeEvento;
                       meo:haScaletta ?scaletta.
              ?scaletta rdf:type meo:Scaletta;
                        owp:hasItem ?brano.
              ?brano rdf:type meo:BranoMusicale;
                     meo:contienePerformanceCanzone meo:Performance` + songName + `.
          } limit 100`;

      //Costruisco la query

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.events = await this.$root.$refs.HelloWorld.makeQuery(query);
      console.log(this.events);
    },
    async findInstruments(songName) {
      this.instruments = [];

      var query = "";

      /*
      * Query per ottenere gli artisti che hanno performato la performance e relativo strumento usato
      */
      query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
        PREFIX owp: <http://www.ontologydesignpatterns.org/cp/owl/bag.owl#>

        select distinct ?nomeArtista ?strumento
        where {
        {
            meo:Performance` + songName + ` meo:performanceSuonataConStrumento ?strumento.
            ?strumento rdf:type meo:StrumentoMusicale;
                    meo:èSuonatoDa ?artista.
            ?artista meo:nomeAgenteMusicale ?nomeArtista.
        }
            union
        {
            meo:Performance` + songName + ` meo:performanceCantataDa ?artista.
            ?artista meo:nomeAgenteMusicale ?nomeArtista.
            }
        } limit 100
`;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.instruments = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>