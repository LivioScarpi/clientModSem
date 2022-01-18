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
        <b-nav-item @click="changePage(7)">Performance Canzoni</b-nav-item>
        <b-nav-item @click="changePage(8)">Strutture Musicali</b-nav-item>
        <b-nav-item @click="changePage(9)">Playlist</b-nav-item>
      </b-navbar-nav>
    </b-navbar>

    <div v-if="this.page === 0">
      <b-card>
        <b-card-title>
          <h3>Musical Events Ontology</h3>
        </b-card-title>
        <b-card-sub-title class="mb-2">
          <h4>L'ontologia degli eventi musicali</h4>
        </b-card-sub-title>

        <div class="p-5 h5">
          <b-card-text>
            Oggigiorno la musica è parte fondamentale di ogni persona. Essa ci accompagna durante i viaggi in macchina
            quando accendiamo la radio, durante lunghe sessioni di lavoro durante le quali ci fa da sottofondo, durante
            le
            faccende domestiche e in moltissimi altri momenti della nostra vita quotidiana. La musica ha caratterizzato
            la
            vita dell’essere umano da sempre, tenendogli compagnia grazie ai vinili, cd e a oggi grazie alle piattaforme
            digitali come Spotify, Apple Music, Amazon Music, ecc. che offrono agli utenti migliaia di migliaia di brani
            e
            moltissime informazioni su tutti gli artisti proprio grazie a delle basi di conoscenza.
          </b-card-text>

          <b-card-text>La musica ha un potere che va oltre l’immaginabile, è in grado di trasportarci in altri posti con
            la mente, di farci rilassare quando siamo stressati, di rievocare ricordi belli e brutti, di farci saltare,
            ballare e divertire come non mai.
          </b-card-text>

          <b-card-text>Le sensazioni che si provano quando parte una delle nostre canzoni preferite, quando sentiamo la
            pelle d’oca attraversarci tutto il corpo, quando cantiamo a squarciagola o quando varchiamo l’ingresso di un
            palazzetto per sentire il concerto live di un nostro cantante preferito sono uniche nel loro genere,
            difficili
            da descrivere a parole.
          </b-card-text>

          <b-card-text>La musica è in grado di stabilire un legame unico tra artista e ascoltatore, il quale arriva a
            considerare l’artista parte della propria famiglia, della propria vita e delle proprie giornate.

          </b-card-text>

          <b-card-text>Ma l’ascolto della musica digitale grazie alle piattaforme moderne non potrà mai suscitare tutte
            quelle sensazioni che nascono quando si vede l’artista o la band suonare dal vivo, a poca distanza rispetto
            a
            dove siamo noi, con la voce tremolante per tutta la folla che urla a squarciagola le sue canzoni. Gli eventi
            live sono l’aspetto più fondamentale in assoluto della musica, lo sono stati fin da sempre, e mai come oggi
            in
            questo periodo di Covid ci sono mancati così tanto i concerti, i festival e tutte le emozioni a essi
            collegate.

          </b-card-text>

          <b-card-text>La ricerca semantica del nostro progetto permette quindi a un utente di consultare informazioni
            riguardanti il dominio degli eventi live: artisti che hanno partecipato ai concerti, città in cui sono stati
            tenuti, struttura musicale che ha ospitato l’evento, canzoni che sono state suonate in scaletta e artisti
            che
            le hanno cantate/suonate in quel determinato live (in quanto un artista può performare una canzone in un
            certo
            live con uno strumento e in un altro live con un altro strumento o non suonarla proprio).

          </b-card-text>

          <b-card-text>È in questo contesto che si colloca il dominio del nostro progetto: essendo appassionati di
            musica
            e amanti degli eventi live riteniamo molto utile fornire all’utente tutte le informazioni necessarie per
            sapere quali sono stati tutti i eventi live a cui gli artisti e le band hanno partecipato, fornendo tutte le
            canzoni suonate a quell’evento, permettendogli di scoprire quali artisti hanno performato le canzoni in
            scaletta e con quali strumenti.
          </b-card-text>
        </div>
        <b-card-footer>
          <b-card-text>Link ad altre pagine utili
          </b-card-text>
          <b-link href="http://xmlns.com/foaf/spec/" class="card-link">FOAF</b-link>
          <b-link href="https://www.dublincore.org/resources/glossary/ontology/" class="card-link">Dublin Core</b-link>
          <b-link href="http://musicontology.com" class="card-link">The Music Ontology</b-link>
          <b-link href="http://motools.sourceforge.net/event/event.122.html#intro" class="card-link">The Event
            Ontology
          </b-link>
          <b-link href="https://www.wikidata.org/wiki/Wikidata:Main_Page" class="card-link">Wikidata</b-link>
        </b-card-footer>


      </b-card>
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
    <div v-else-if="this.page === 7">
      <PerformanceCanzoni/>
    </div>
    <div v-else-if="this.page === 8">
      <StruttureMusicali/>
    </div>
    <div v-else-if="this.page === 9">
      <Playlist/>
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
import PerformanceCanzoni from "@/components/PerformanceCanzoni";
import StruttureMusicali from "./StruttureMusicali";
import Playlist from "./Playlist";

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
    Canzoni,
    PerformanceCanzoni,
    StruttureMusicali,
    Playlist
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
  },

  methods: {
    changePage(page) {
      this.page = page;
      console.log(this.page);
    },
    foo: function () {
      alert('this is A.foo');
    },

    /*
    * Metodo che si occupa di contattare l'endpoint indicato da endpointUrl, di elaborare la risposta ottenuta
    * e riorganizzarla in una struttura adatta per essere utilizzata con le b-table di bootstrap
    */
    async makeQuery(query) {
      var queryData = [];

      const client = new SparqlClient({endpointUrl})
      const stream = await client.query.select(query)

      var i = 0;
      var nrow = 0;

      /* Recupero il numero di righe della nostra risposta */
      stream.on('data', () => {
        nrow++;
      })

      /* Inizializzo l'oggetto da passare al component BTable */
      queryData = new Array(Object.entries(nrow).length);

      stream.on('data', row => {
        console.log(row);

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
