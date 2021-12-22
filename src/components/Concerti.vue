<template>
  <div class="m-3">
    <div v-if="artistsLoaded">
      <!--
      <b-form-select v-model="artistSelected" :options="allArtists"></b-form-select>
      <div class="mt-3">Selected: <strong>{{ artistSelected }}</strong></div>
      <div>
        <b-button variant="primary" @click="findLiveEvents()">Trova eventi Live dell'artista</b-button>
      </div>-->
      <h3>Eventi live degli artisti</h3>


      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-form-checkbox
                id="checkbox-1"
                v-model="status"
                name="checkbox-1"
                value="selected"
                unchecked-value="not_selected"
            >
              Mostra solo concerti in cui l'artista ha avuto il ruolo di artista principale
            </b-form-checkbox>
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allArtists" :key="index" @click="findLiveEvents(elem.value)" button>{{ elem.value }}</b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
              <div v-if="artistsevents.length > 0">
                <b-table striped hover :items="artistsevents"></b-table>
              </div>
            <div v-else>
              <h5>L'artista non ha partecipato a nessun concerto con il ruolo di artista principale</h5>
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
  name: "Concerti",

  components: {
    //BButton,
    BContainer, BRow, BCol
  },

  data: function () {
    return {
      artistSelected: null,
      allArtists: [],
      artistsLoaded: false,
      allLoaded: false,
      artistsevents: [],
      status: 'not_selected'
    }
  },

  async mounted() {

    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select ?nomeArtista
      where {
        ?artista rdf:type meo:AgenteMusicale;
        meo:nomeAgenteMusicale ?nomeArtista.
      } limit 100
    `;

    const client = new SparqlClient({endpointUrl})
    const stream = await client.query.select(query)

    var self = this;

    stream.on('data', row => {
      console.log(row);
      Object.entries(row).forEach(([key, value]) => {

        var artistName = value.value.replace('http://www.modsem.org/musicalEventsOntology#', '');

        self.allArtists.push({value: artistName, text: artistName});

        console.log(`${key}: ${value.value} (${value.termType})`)
      })

      self.artistsLoaded = true;
    })

    stream.on('error', err => {
      console.error(err)
    })
  },

  methods: {
    async findLiveEvents(artistName) {
      this.artistsevents = [];

      var query = "";

      if(this.status === 'selected') {
        query = `
PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

select distinct ?artista ?evento ?nomeEvento ?dataEvento ?capienzaMassimaEvento ?strutturaEvento
where {
    {
    ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    ?evento rdf:type meo:Concerto;
            meo:nomeEventoMusicale ?nomeEvento;
            meo:dataEventoMusicale ?dataEvento;
            meo:eventoInStruttura ?strutturaEvento;
            meo:capienzaMassimaEventoMusicale ?capienzaMassimaEvento.
            FILTER NOT EXISTS {
    ?artista meo:agentePartecipaApertura ?evento.
    }

    }
} limit 100
      `;
      } else {
        query = `
PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

select distinct ?artista ?evento ?nomeEvento ?dataEvento ?capienzaMassimaEvento ?strutturaEvento
where {
    {
    ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    ?evento rdf:type meo:Concerto;
            meo:nomeEventoMusicale ?nomeEvento;
            meo:dataEventoMusicale ?dataEvento;
            meo:eventoInStruttura ?strutturaEvento;
            meo:capienzaMassimaEventoMusicale ?capienzaMassimaEvento.

    }
} limit 100
      `;
      }

      //Costruisco la query

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.artistsevents = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>