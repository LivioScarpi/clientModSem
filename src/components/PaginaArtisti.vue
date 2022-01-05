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


      <b-container class="bv-example-row mt-5">
        <b-row class="text-center">
          <b-col cols="3">
            <b-list-group>
              <b-list-group-item v-for="(elem, index) in allArtists" :key="index" @click="callQueryMethods(elem.value)"
                                 button>{{ elem.value }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col cols="9">
            <div class="pb-5">
              <h4>Genere/i musicale/i dell'artista</h4>
              <div v-if="genres.length > 0">
                <b-table striped hover :items="genres"></b-table>
              </div>
              <div v-else>
                <h5>NOTA</h5>
              </div>
            </div>

            <div class="pb-5">
              <h4>Etichetta musicale dell'artista</h4>
              <div v-if="labels.length > 0">
                <b-table striped hover :items="labels"></b-table>
              </div>
              <div v-else>
                <h5>NOTA</h5>
              </div>
            </div>

            <div class="pb-5">
              <h4>Eventi in cui l'artista ha partecipato: MusicalEventsOntology</h4>
              <div v-if="artistsevents.length > 0">
                <b-table striped hover :items="artistsevents"></b-table>
              </div>
              <div v-else>
                <h5>L'artista non ha partecipato a nessun evento</h5>
              </div>
            </div>

            <div class="pb-5">
              <h4>Eventi in cui l'artista ha partecipato: WikiData</h4>
              <div v-if="artistseventswd.length > 0">
                <b-table striped hover :items="artistseventswd"></b-table>
              </div>
              <div v-else>
                <h5>L'artista non ha partecipato a nessun evento secondo WikiData</h5>
              </div>
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
  name: "PaginaArtisti",

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
      genreandlabels: [],
      genres: [],
      labels: [],
      artistseventswd: [], //eventi dell'artista recuperati da wikidata
    }
  },

  async mounted() {
    console.log("ESEGUO QUERY");

    const query = `
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
      PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
      PREFIX owl: <http://www.w3.org/2002/07/owl#>
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

      select distinct ?nomeArtista
      where {
        ?artista rdf:type meo:AgenteMusicale;
        meo:nomeAgenteMusicale ?nomeArtista.
      } limit 100
    `;
    console.log(query);

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

    console.log("ECCOmi");
  },

  methods: {
    callQueryMethods(songName) {
      console.log("sono qua 1");
      this.findLiveEvents(songName);
      //this.findGenreAndLabel(songName);
      //this.findLiveEventsWD(songName);
    },
    async findLiveEvents(artistName) {
      this.artistsevents = [];

      console.log("sono qua");

      //Costruisco la query
      const query = `
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
    ?evento rdf:type meo:EventoMusicale;
            meo:nomeEventoMusicale ?nomeEvento;
            meo:dataEventoMusicale ?dataEvento;
            meo:eventoInStruttura ?strutturaEvento;
            meo:capienzaMassimaEventoMusicale ?capienzaMassimaEvento.
    ?artista meo:agentePartecipaEvento ?evento.
    }
    union
    {
            ?artista rdf:type meo:Artista;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    ?evento rdf:type meo:EventoMusicale;
            meo:nomeEventoMusicale ?nomeEvento;
            meo:dataEventoMusicale ?dataEvento;
            meo:eventoInStruttura ?strutturaEvento;
            meo:capienzaMassimaEventoMusicale ?capienzaMassimaEvento.
    ?artista meo:membroBandPartecipaEvento ?evento.
    }
} limit 100
      `;

      console.log(query);

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.artistsevents = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
    async findGenreAndLabel(artistName) {
      this.genres = [];

      //Costruisco la query

      /* OLD:
      * PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX       wdt:  <http://www.wikidata.org/prop/direct/>
PREFIX        wd:  <http://www.wikidata.org/entity/>
PREFIX  wikibase:  <http://wikiba.se/ontology#>
PREFIX        bd:  <http://www.bigdata.com/rdf#>

select distinct ?etichettaGenere
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P264 ?etichetta;
        	wdt:P136 ?genere.
        SERVICE wikibase:label {
            bd:serviceParam wikibase:language "it".
            ?etichetta rdfs:label ?label.
            ?genere rdfs:label ?etichettaGenere
        }
    }
} limit 100*/

      /* ALTRA VERSIONE FUNZIONANTE:
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX       wdt:  <http://www.wikidata.org/prop/direct/>
PREFIX        wd:  <http://www.wikidata.org/entity/>
PREFIX  wikibase:  <http://wikiba.se/ontology#>
PREFIX        bd:  <http://www.bigdata.com/rdf#>

select distinct ?etichettaLabel ?genereLabel
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "Pinguini Tattici Nucleari").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P264 ?etichetta;
        	wdt:P136 ?genere.
        ?etichetta rdfs:label ?etichettaLabel .
        ?genere rdfs:label ?genereLabel .
        FILTER (langMatches( lang(?etichettaLabel), "it" ) )
        FILTER (langMatches( lang(?genereLabel), "it" ) )
    }
} limit 1
      * */

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

select distinct ?genereLabel
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "Pinguini Tattici Nucleari").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P136 ?genere.
        ?genere rdfs:label ?genereLabel .
        FILTER (langMatches( lang(?genereLabel), "it" ) )
    }
} limit 1
      `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.genres = await this.$root.$refs.HelloWorld.makeQuery(query);

      this.labels = [];

      /* OLD:
      PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX       wdt:  <http://www.wikidata.org/prop/direct/>
PREFIX        wd:  <http://www.wikidata.org/entity/>
PREFIX  wikibase:  <http://wikiba.se/ontology#>
PREFIX        bd:  <http://www.bigdata.com/rdf#>

select distinct ?label
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P264 ?etichetta;
        	wdt:P136 ?genere.
        SERVICE wikibase:label {
            bd:serviceParam wikibase:language "it".
            ?etichetta rdfs:label ?label.
            ?genere rdfs:label ?etichettaGenere
        }
    }
} limit 100
       */

      const query1 = `
PREFIX meo: <http://www.modsem.org/musicalEventsOntology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX       wdt:  <http://www.wikidata.org/prop/direct/>
PREFIX        wd:  <http://www.wikidata.org/entity/>
PREFIX  wikibase:  <http://wikiba.se/ontology#>
PREFIX        bd:  <http://www.bigdata.com/rdf#>

select distinct ?etichettaLabel
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P264 ?etichetta.
        ?etichetta rdfs:label ?etichettaLabel .
        FILTER (langMatches( lang(?etichettaLabel), "it" ) )
    }
} limit 1
      `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.labels = await this.$root.$refs.HelloWorld.makeQuery(query1);

    },

    async findLiveEventsWD(artistName) {
      this.artistseventswd = [];

      //Costruisco la query
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

select ?nomeArtista ?nomeEvento
where {
            ?artista rdf:type meo:AgenteMusicale;
             meo:nomeAgenteMusicale ?nomeArtista.
    FILTER regex(?nomeArtista, "` + artistName + `").
    service <https://query.wikidata.org/sparql> {
        ?artista wdt:P1344 ?evento.
        SERVICE wikibase:label {
            bd:serviceParam wikibase:language "it".
            ?evento rdfs:label ?nomeEvento.
        }
    }
} limit 100
      `;

      //Chiamo il metodo che esegue la query: situato nel component principale
      this.artistseventswd = await this.$root.$refs.HelloWorld.makeQuery(query);
    },
  }
}

</script>

<style scoped>

</style>