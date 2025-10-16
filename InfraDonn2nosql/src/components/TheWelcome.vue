<script setup lang="ts">
import WelcomeItem from './WelcomeItem.vue'
import DocumentationIcon from './icons/IconDocumentation.vue'
import ToolingIcon from './icons/IconTooling.vue'
import EcosystemIcon from './icons/IconEcosystem.vue'
import CommunityIcon from './icons/IconCommunity.vue'
import SupportIcon from './icons/IconSupport.vue'
import { onMounted, ref } from 'vue';
import PouchDB from 'pouchdb'

declare interface Post {
  post_name: string;
  post_content: string;
  attributes: {
    creation_date: any;
  };
}

const counter = ref(0);

const increment = () => {
  counter.value++;
}

// Référence à la base de données
const storage = ref()
// Données stockées
const postsData = ref<Post[]>([])

// Initialisation de la base de données
const initDatabase = () => {
  console.log('=> Connexion à la base de données');
  const db = new PouchDB('http://aidan:nadia@localhost:5984/database')
  if (db) {
    console.log("Connecté à la collection : " + db?.name)
    storage.value = db
  } else {
    console.warn('Echec lors de la connexion à la base de données')
  }
}

// Récupération des données
const fetchData = () => {
  // TODO Récupération des données
  // https://pouchdb.com/api.html#batch_fetch
  // Regarder l'exemple avec function allDocs
  // Remplir le tableau postsData avec les données récupérées
  storage.value.allDocs({
    include_docs: true,
    attachments: true
  }).then(function (result) {
    // handle result
    console.log(result)
  }).catch(function (err) {
    console.log(err);
  });
}

const addDocument = () => {
  storage.value.put({
    _id: 'mydoc',
    title: 'Heroes'
  }).then(function (response) {
    // handle response
    console.log(response)
  }).catch(function (err) {
    console.log(err);
  });
}

const updateDocument = () => {
  storage.value.get('mydoc').then(function (doc) {
    return storage.value.put({
      _id: 'mydoc',
      _rev: doc._rev,
      title: "Let's Dance"
    });
  }).then(function (response) {
    // handle response
    console.log(response)
  }).catch(function (err) {
    console.log(err);
  });
}

const deleteDocument = () => {
  storage.value.get('mydoc').then(function (doc) {
    return storage.value.remove(doc);
  }).then(function (result) {
    // handle result
    console.log(result)
  }).catch(function (err) {
    console.log(err);
  });
}

onMounted(() => {
  console.log('=> Composant initialisé');
  initDatabase()
  fetchData()
  //addDocument()
  //updateDocument()
  //deleteDocument()
});

const openReadmeInEditor = () => fetch('/__open-in-editor?file=README.md')
</script>

<template>
  <h1>Fetch Data</h1>
  <article v-for="post in postsData" v-bind:key="(post as any).id">
    <h2>{{ post.post_name }}</h2>
    <p>{{ post.post_content }}</p>
  </article>

  <p>Counter: {{ counter }}</p>
  <button @click="increment">+1</button>
</template>
