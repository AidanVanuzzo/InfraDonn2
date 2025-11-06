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
let post_name = ""
let post_content = ""

const increment = () => {
  counter.value++;
}

// Référence à la base de données
const storage = ref()
// Données stockées
let postsData = ref<Post[]>([])

// Initialisation de la base de données
const initDatabase = () => {
  console.log('=> Connexion à la base de données');
  //const db = new PouchDB('http://aidan:nadia@localhost:5984/database')
  const db = new PouchDB('tuto')
  PouchDB.replicate('http://aidan:nadia@localhost:5984/database','tuto')
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
  }).then((result) => {
    // handle result
    postsData.value = result.rows.map((row) => row.doc);
    console.log(result)
  }).catch(function (err) {
    console.log(err);
  });
}

const addDocument = () => {
  console.log(post_name)
  storage.value.post({
    post_name: post_name,
    post_content: post_content
  }).then((response) => {
    // handle response
    console.log(response);
    fetchData();
  }).catch(function (err) {
    console.log(err);
  });
}

const updateDocument = (post) => {

  console.log(post)

  const updatedPost = {...post, post_name : "laurent" }
  console.log(updatedPost)

  storage.value.put(updatedPost);

  return;
  storage.value.get('mydoc').then(function (doc) {
    return storage.value.put({
      _id: 'mydoc',
      _rev: doc._rev,
      title: "Let's Dance"
    });
  }).then(function (response) {
    // handle response
    console.log(response);
    fetchData();
  }).catch(function (err) {
    console.log(err);
  });
}

const deleteDocument = (post) => {

  console.log(post)

  storage.value.remove(post);

  return;
  storage.value.get(post).then(function (post) {
    return storage.value.remove(post);
  }).then(function (result) {
    // handle result
    console.log(result);
    fetchData();
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
  <h1>Add Post</h1>
  <input type="text" id="postName" name="postName" v-model="post_name" placeholder="post name" />
  <input type="text" id="postContent" name="postContent" v-model="post_content" placeholder="post content" />
  <button @click="addDocument()">Add Document</button>
  <h1>Fetch Data</h1>
  <article v-for="post in postsData" v-bind:key="(post as any).id">
    <h2>{{ post.post_name }}</h2>
    <p>{{ post.post_content }}</p>
    <button @click="updateDocument(post)">Update Document</button>
  <button @click="deleteDocument(post)">Delete Document</button>
  
  </article>

  <p>Counter: {{ counter }}</p>
  <button @click="increment">+1</button>
  <p>{{ post_name }}</p>
  <p>{{ post_content }}</p>
</template>
