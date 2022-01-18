<template>
  <div class="m-3">
    <div v-if="groupsLoaded">
      <h3>Playlist con le canzoni degli eventi</h3>

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

            <h4>Canzoni contenute nella playlst selezionata</h4>
            <div v-if="playlistSongs.length > 0">
              <b-table striped hover :items="playlistSongs"></b-table>
            </div>
            <div v-else>
              <h5>Non ci sono canzoni in questa playlist</h5>
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
      playlistSongs: [],
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

      select distinct ?nomePlaylist
      where {
        ?playlist rdf:type meo:Playlist;
        meo:nomePlaylist ?nomePlaylist.
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
      this.findLiveEvents(songName);
    },
    async findLiveEvents(playlistName) {
      this.playlistSongs = [];

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

        select ?nomeCanzone ?nomeEvento
        where {
            ?playlist rdf:type meo:Playlist;
                      meo:nomePlaylist ?nomePlaylist;
                      meo:playlistContieneCanzone ?canzone;
                      meo:playlistAssociataEvento ?evento.
            ?evento meo:nomeEventoMusicale ?nomeEvento.
            ?canzone meo:titoloCanzone ?nomeCanzone.
            FILTER regex(?nomePlaylist, "` + playlistName + `")
        } limit 100
    `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.playlistSongs = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>