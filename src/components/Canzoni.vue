<template>
  <div class="m-3">
    <div v-if="artistsLoaded">
      <h3>Canzoni</h3>

      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allEvents" :key="index" @click="callQueryMethods(elem.value)"
                                 button>{{ elem.value }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
            <h4>Eventi in cui la canzone era in scaletta</h4>
            <div v-if="events.length > 0">
              <b-table striped hover :items="events"></b-table>
            </div>
            <div v-else>
              <h5>La canzone non è stata suonata o cantata in nessun evento</h5>
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
  name: "Canzoni",

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
    * Query per ottenere tutte le canzoni
    */
    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select ?titoloCanzone
      where {
          ?canzone rdf:type meo:Canzone;
                  meo:titoloCanzone ?titoloCanzone.
      } limit 100
    `;

    const client = new SparqlClient({endpointUrl})
    const stream = await client.query.select(query)

    var self = this;

    stream.on('data', row => {
      console.log(row);
      Object.entries(row).forEach(([key, value]) => {

        var artistName = value.value.replace('http://www.modsem.org/musicalEventsOntology#', '');

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
      * Query per ottenere tutti i gli eventi in cui una canzone è stata performata
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
                     meo:contienePerformanceCanzone ?performanceCanzone.
              ?performanceCanzone rdf:type meo:PerformanceCanzone;
                       meo:performanceDiCanzone ?canzone.
                ?canzone rdf:type meo:Canzone;
                       meo:titoloCanzone ?titoloCanzone.
              FILTER regex(?titoloCanzone, "` + songName + `").
          } limit 100
`;

      //Costruisco la query

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.events = await this.$root.$refs.HelloWorld.makeQuery(query);
      console.log(this.events);
    },
    async findInstruments(songName) {
      this.instruments = [];

      var query = "";

      //Tutti gli eventi in cui una canzone è in scaletta
      query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
        PREFIX owp: <http://www.ontologydesignpatterns.org/cp/owl/bag.owl#>

        select distinct ?canzone ?strumento ?nomeArtista
        where {
            ?canzone rdf:type meo:Canzone;
                     meo:titoloCanzone ?titoloCanzone;
                meo:canzoneSuonataConStrumento ?strumento.
            ?strumento rdf:type meo:StrumentoMusicale;
                    meo:èSuonatoDa ?artista.
            ?artista meo:nomeAgenteMusicale ?nomeArtista.
            FILTER regex(?titoloCanzone, "` + songName + `").
        } limit 100`;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.instruments = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>