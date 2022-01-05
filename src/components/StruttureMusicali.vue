<template>
  <div class="m-3">
    <div v-if="groupsLoaded">
      <!--
      <b-form-select v-model="artistSelected" :options="allGroups"></b-form-select>
      <div class="mt-3">Selected: <strong>{{ artistSelected }}</strong></div>
      <div>
        <b-button variant="primary" @click="findLiveEvents()">Trova eventi Live dell'artista</b-button>
      </div>-->
      <h3>StruttureMusicali</h3>

      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allGroups" :key="index" @click="callQueryMethods(elem.value)" button>{{ elem.value }}</b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">

            <h4>Eventi tenuti in questa struttura</h4>
            <div v-if="events.length > 0">
              <b-table striped hover :items="events"></b-table>
            </div>
            <div v-else>
              <h5>Non ci sono informazioni su questa città</h5>
            </div>

            <h4>Informazioni su questa struttura</h4>
            <div v-if="structureInfo.length > 0">
            <b-table striped hover :items="structureInfo"></b-table>
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
  name: "StruttureMusicali",

  components: {
    //BButton,
    BContainer, BRow, BCol
  },

  data: function () {
    return {
      artistSelected: null,
      allGroups: [],
      groupsLoaded: false,
      structureInfo: [],
      events: [],
    }
  },

  async mounted() {

    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select distinct ?nomeStruttura
      where {
        ?struttura rdf:type meo:StrutturaMusicale;
        meo:nomeStruttura ?nomeStruttura.
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
      this.findEvents(songName);
      this.findInfo(songName);
    },
    async findInfo(structureName) {
      this.structureInfo = [];

      //Costruisco la query
      const query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        select distinct ?nomeStruttura ?capienzaMassima ?nomeCittà
        where {
            ?struttura rdf:type meo:StrutturaMusicale;
            meo:nomeStruttura ?nomeStruttura;
            meo:strutturaCollocataInCittà ?città;
            meo:capienzaMassimaStruttura ?capienzaMassima.
            FILTER regex(?nomeStruttura, "` + structureName+ `").
    		?città rdf:type meo:Città;
             		meo:nomeCittà ?nomeCittà.
        } limit 100
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.structureInfo = await this.$root.$refs.HelloWorld.makeQuery(query);
    },

    async findEvents(structureName) {
      this.events = [];

      //Costruisco la query
      const query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        select distinct ?nomeEvento ?tipoEvento ?dataEventoMusicale ?capienzaMassimaEventoMusicale ?capienzaMassimaStruttura ?nomeCitt
        where {
            ?struttura rdf:type meo:StrutturaMusicale;
             meo:nomeStruttura ?nomeStruttura;
            meo:strutturaCollocataInCittà ?città;
            meo:capienzaMassimaStruttura ?capienzaMassimaStruttura;
            meo:strutturaHaEvento ?evento.
            ?città rdf:type meo:Città;
                    meo:nomeCittà ?nomeCittà.
            ?evento rdf:type meo:EventoMusicale;
                    meo:nomeEventoMusicale ?nomeEvento;
                    meo:capienzaMassimaEventoMusicale ?capienzaMassimaEventoMusicale;
                    meo:tipoEventoMusicale ?tipoEvento;
                    meo:dataEventoMusicale ?dataEventoMusicale.
            FILTER regex(?nomeStruttura, "` + structureName+ `").
        } limit 100
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.events = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>