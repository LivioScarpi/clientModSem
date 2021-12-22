<template>
  <div class="m-3">
    <div v-if="groupsLoaded">
      <!--
      <b-form-select v-model="artistSelected" :options="allGroups"></b-form-select>
      <div class="mt-3">Selected: <strong>{{ artistSelected }}</strong></div>
      <div>
        <b-button variant="primary" @click="findLiveEvents()">Trova eventi Live dell'artista</b-button>
      </div>-->
      <h3>Componenti dei gruppi</h3>


      <b-container class="bv-example-row">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allGroups" :key="index" @click="findLiveEvents(elem.value)" button>{{ elem.value }}</b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
            <div v-if="groupComponents.length > 0">
            <b-table striped hover :items="groupComponents"></b-table>
            </div>
            <div v-else>
              <h5>Il gruppo non ha nessun componente</h5>
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
  name: "Gruppi",

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
    }
  },

  async mounted() {

    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select distinct ?nomeGruppo
      where {
        ?artista rdf:type meo:Gruppo;
        meo:nomeAgenteMusicale ?nomeGruppo.
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
    async findLiveEvents(groupName) {
      this.groupComponents = [];

      //Costruisco la query
      const query = `
        PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
        PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        PREFIX owl: <http://www.w3.org/2002/07/owl#>
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

        select distinct ?nomeArtista ?nomeGruppo
        where {
            {
                #artisti che fanno parte di una band
                ?artista rdf:type meo:AgenteMusicale;
                         meo:nomeAgenteMusicale ?nomeArtista.
                ?gruppo rdf:type meo:Gruppo;
                      meo:formataDa ?artista;
                      meo:nomeAgenteMusicale ?nomeGruppo;
                      FILTER regex(?nomeGruppo, "` + groupName + `").
            }
            union
            {
                #artisti che fanno parte di un duo
                ?artista rdf:type meo:AgenteMusicale;
                         meo:nomeAgenteMusicale ?nomeArtista.
                ?gruppo rdf:type meo:Gruppo;
                      meo:duoFormatoDa ?artista;
                      meo:nomeAgenteMusicale ?nomeGruppo;
                      FILTER regex(?nomeGruppo, "` + groupName + `").
            }
        } limit 100
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.groupComponents = await this.$root.$refs.HelloWorld.makeQuery(query);
    },

  }
}

</script>

<style scoped>

</style>