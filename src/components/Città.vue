<template>
  <div class="m-3">
    <div v-if="groupsLoaded">
      <h3>Eventi nelle città</h3>

      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allGroups" :key="index" @click="callQueryMethods(elem.value)"
                                 button>{{ elem.value }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">

            <h4>informazioni sulla città: WikiData</h4>
            <div v-if="cityInfo.length > 0">
              <b-table striped hover :items="cityInfo"></b-table>
            </div>
            <div v-else>
              <h5>Non ci sono informazioni su questa città</h5>
            </div>

            <h4>Eventi tenuti in questa città</h4>
            <div v-if="groupComponents.length > 0">
              <b-table striped hover :items="groupComponents"></b-table>
            </div>
            <div v-else>
              <h5>In questa città non ci sono stati eventi</h5>
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
  name: "Città",

  components: {
    //BButton,
    BContainer, BRow, BCol
  },

  data: function () {
    return {
      artistSelected: null,
      allGroups: [],
      groupsLoaded: false,
      groupComponents: [],
      cityInfo: [],
    }
  },

  async mounted() {

    /*
    * Query per ottenere tutte le città
    */
    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select distinct ?nomeCittà
      where {
        ?città rdf:type meo:Città;
        meo:nomeCittà ?nomeCittà.
      } limit 100
    `;

    const client = new SparqlClient({endpointUrl})
    const stream = await client.query.select(query)

    var self = this;

    stream.on('data', row => {
      console.log(row);
      Object.entries(row).forEach(([key, value]) => {

        var artistName = value.value.replace('http://www.modsem.org/musicalEventsOntology#', '');

        self.allGroups.push({value: artistName, text: artistName});

        console.log(`${key}: ${value.value} (${value.termType})`)
      })

      self.groupsLoaded = true;
    })

    stream.on('error', err => {
      console.error(err)
    })
  },

  methods: {
    callQueryMethods(songName) {
      this.findCityInfo(songName);
      this.findLiveEvents(songName);
    },
    async findLiveEvents(cityName) {
      this.groupComponents = [];

      //Costruisco la query

      /*
      * Query per ottenere gli eventi tenuti in una città
      */
      const query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        select distinct ?nomeEvento ?nomeStruttura ?nomeCittà
        where {
            ?città rdf:type meo:Città;
            meo:nomeCittà ?nomeCittà.
            FILTER regex(?nomeCittà, "` + cityName + `").
          ?struttura rdf:type meo:StrutturaMusicale;
            meo:strutturaCollocataInCittà ?città;
            meo:nomeStruttura ?nomeStruttura.
          ?evento rdf:type meo:EventoMusicale;
            meo:eventoInStruttura ?struttura;
            meo:nomeEventoMusicale ?nomeEvento.
        } limit 100
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.groupComponents = await this.$root.$refs.HelloWorld.makeQuery(query);
    },

    async findCityInfo(cityName) {
      this.cityInfo = [];

      //Costruisco la query


      /*
      * Query per ottenere informazioni aggiuntive sulla città, contattando l'endpoint SPARQL di Wikidata
      */
      const query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
        PREFIX       wdt:  <http://www.wikidata.org/prop/direct/>
        PREFIX        wd:  <http://www.wikidata.org/entity/>
        PREFIX  wikibase:  <http://wikiba.se/ontology#>
        PREFIX        bd:  <http://www.bigdata.com/rdf#>

        select ?nomeCittà ?abbreviazione ?popolazione ?codicePostale
        where {
                    ?città rdf:type meo:Città;
                     meo:nomeCittà ?nomeCittà.
            FILTER regex(?nomeCittà, "` + cityName + `").
            service <https://query.wikidata.org/sparql> {
                ?città wdt:P281 ?codicePostale;
                       wdt:P395 ?abbreviazione;
                       wdt:P1082 ?popolazione.
                SERVICE wikibase:label {
                    bd:serviceParam wikibase:language "it".
                    ?codicePostale rdfs:label ?postalCodelabel.
                    ?abbreviazione rdfs:label ?abbreviazionelabel.
                    ?popolazione rdfs:label ?popolazione
                }
            }
        } limit 1
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.cityInfo = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>