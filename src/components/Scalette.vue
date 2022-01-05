<template>
  <div class="m-3">
    <div v-if="artistsLoaded">
      <!--
      <b-form-select v-model="eventSelected" :options="allEvents"></b-form-select>
      <div class="mt-3">Selected: <strong>{{ eventSelected }}</strong></div>
      <div>
        <b-button variant="primary" @click="findLiveEvents()">Trova eventi Live dell'artista</b-button>
      </div>-->
      <h3>Scalette degli eventi</h3>
      <h4>NB! Se nella colonna "Strumento" non è presente nulla, vuol dire che l'artista ha cantato quella canzone</h4>


      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allEvents" :key="index" @click="findLiveEvents(elem.value)" button>{{ elem.value }}</b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
              <div v-if="events.length > 0">
                <b-table striped hover :items="events"></b-table>
              </div>
            <div v-else>
              <h5>L'evento non ha nessuna canzone in scaletta</h5>
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
  name: "Scalette",

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
      status: 'not_selected',
    }
  },

  async mounted() {

    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select ?nomeEvento
      where {
          ?evento rdf:type meo:EventoMusicale;
                  meo:nomeEventoMusicale ?nomeEvento.
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
    async findLiveEvents(eventName) {
      this.events = [];

      var query = "";

      /*        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
        PREFIX owp: <http://www.ontologydesignpatterns.org/cp/owl/bag.owl#>


        select ?titoloCanzone ?nomeEvento
        where {
            ?evento  rdf:type meo:EventoMusicale;
                     meo:nomeEventoMusicale ?nomeEvento;
                     meo:haScaletta ?scaletta.
            ?scaletta rdf:type meo:Scaletta;
                 owp:hasItem ?brano.
            ?brano rdf:type meo:BranoMusicale;
                   meo:contieneCanzone ?canzone.
            ?canzone rdf:type meo:Canzone;
                    meo:titoloCanzone ?titoloCanzone.
            FILTER regex(?nomeEvento, "` + eventName + `").
        } limit 100*/

        query = `
PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX owp: <http://www.ontologydesignpatterns.org/cp/owl/bag.owl#>


select distinct ?titoloCanzone ?performanceCanzone ?nomeArtista ?strumento
where {
    {
    ?evento  rdf:type meo:EventoMusicale;
             meo:nomeEventoMusicale ?nomeEvento;
             meo:haScaletta ?scaletta.
    ?scaletta rdf:type meo:Scaletta;
              owp:hasItem ?brano.
    ?brano rdf:type meo:BranoMusicale;
           meo:contienePerformanceCanzone ?performanceCanzone.
    ?performanceCanzone rdf:type meo:PerformanceCanzone;
                        #meo:performancePerformataDa ?artista;
                        meo:performanceSuonataConStrumento ?strumento;
                        meo:performanceDiCanzone ?canzone.
    ?strumento rdf:type meo:StrumentoMusicale;
               meo:èSuonatoDa ?artista.
    ?canzone rdf:type meo:Canzone;
             meo:titoloCanzone ?titoloCanzone.
    ?artista meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeEvento, "` + eventName + `").
    }
    union
    {
            ?evento  rdf:type meo:EventoMusicale;
             meo:nomeEventoMusicale ?nomeEvento;
             meo:haScaletta ?scaletta.
    ?scaletta rdf:type meo:Scaletta;
              owp:hasItem ?brano.
    ?brano rdf:type meo:BranoMusicale;
           meo:contienePerformanceCanzone ?performanceCanzone.
    ?performanceCanzone rdf:type meo:PerformanceCanzone;
                        meo:performanceCantataDa ?artista;
                        meo:performanceDiCanzone ?canzone.
    ?canzone rdf:type meo:Canzone;
             meo:titoloCanzone ?titoloCanzone.
    ?artista meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeEvento, "` + eventName + `").
    }
} limit 100

`;

      //Costruisco la query

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.events = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>