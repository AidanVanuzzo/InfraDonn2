<script setup lang="ts">
import WelcomeItem from './WelcomeItem.vue'
import DocumentationIcon from './icons/IconDocumentation.vue'
import ToolingIcon from './icons/IconTooling.vue'
import EcosystemIcon from './icons/IconEcosystem.vue'
import CommunityIcon from './icons/IconCommunity.vue'
import SupportIcon from './icons/IconSupport.vue'
import { onMounted, ref } from 'vue';
import PouchDB from 'pouchdb';
import findPlugin from "pouchdb-find";
PouchDB.plugin(findPlugin);

//interfaces
declare interface Post {
  _id?: string,
  post_name: string;
  post_content: string;
  post_like: number;
  attributes: {
    creation_date: any;
  };
}

declare interface Comment {
  comment_content: string;
  post_id: string;
  attributes: {
    creation_date: any;
  }
}

//databases
const post = 'http://aidan:nadia@localhost:5984/post_vanuzzo_aidan'
const comment = 'http://aidan:nadia@localhost:5984/comment_vanuzzo_aidan'

//
let post_name = ""
let post_content = ""
let comment_content = ""

// Référence à la base de données
const storagePost = ref()
const storageComment = ref()
// Données stockées
let postsData = ref<Post[]>([])
let commentsData = ref<Comment[]>([])

const dataCreateIndex = () => {
  storageComment.value.createIndex({
    index: {
      fields: ['Comment.post_id']
    }
  }).then((response: any) => {
    console.log(response);
  }).catch((err: any) => {
    console.log(err);
  })
}

// Initialisation de la base de données
const initDatabase = () => {
  console.log('=> Connexion à la base de données');
  //const db = new PouchDB('http://aidan:nadia@localhost:5984/post')
  const dbPost = new PouchDB('post')
  const dbComment = new PouchDB('comment')
  PouchDB.replicate(post, 'post')
  PouchDB.replicate(comment, 'comment')
  if (dbPost) {
    console.log("Connecté à la collection : " + dbPost?.name)
    storagePost.value = dbPost
  } else {
    console.warn('Echec lors de la connexion à la base de données Post')
  }
  if (dbComment) {
    console.log("Connecté à la collection : " + dbComment?.name)
    storageComment.value = dbComment
  } else {
    console.warn('Echec lors de la connexion à la base de données Comment')
  }
}

// Récupération des données
const fetchData = () => {
  // TODO Récupération des données
  // https://pouchdb.com/api.html#batch_fetch
  // Regarder l'exemple avec function allDocs
  // Remplir le tableau postsData avec les données récupérées
  storagePost.value.allDocs({
    include_docs: true,
    attachments: true
  }).then((result: { rows: any[] }) => {
    // mettre uniquement post_content
    postsData.value = result.rows.map((row: { doc: any }) => row.doc).filter((doc: { post_content: any }) => !!doc.post_content);
  }).catch(function (err: any) {
    console.log(err);
  });

  storageComment.value.allDocs({
    include_docs: true,
    attachments: true
  }).then((result: { rows: any[] }) => {
    commentsData.value = result.rows.map((row: { doc: any }) => row.doc).filter((doc: { comment_content: any }) => !!doc.comment_content);
  }).catch((err: any) => {
    console.log(err);
  })
}

const addDocument = () => {
  console.log(post_name)
  storagePost.value.post({
    post_name: post_name,
    post_content: post_content,
    post_like: 0
  }).then((response: any) => {
    // handle response
    console.log(response);
    //location.reload();
    fetchData();
    post_content = ""
    post_name = ""
  }).catch((err: any) => {
    console.log(err);
  });
}

const addComment = (post: any) => {
  storageComment.value.post({
    comment_content: comment_content,
    post_id: post._id
  }).then((response: any) => {
    console.log(post._id);
    fetchData();
    comment_content = ""
  }).catch((err: any) => {
    console.log(err);
  })
}

const addLike = (post: any) => {

  console.log(post.post_name)

  let post_like = post.post_like;

  post_like++

  const likedPost = {
    ...post,
    post_like: post_like
  }

  console.log(likedPost)

  storagePost.value.put(likedPost).then((response: any) => {
    console.log(response);
    fetchData();
  }).catch((err: any) => {
    console.log(err);
  })

  return;
}

let post_name_change = ""
let post_content_change = ""

const updateDocument = (post: any) => {

  console.log(post.post_name)

  const updatedPost = {
    ...post,
    post_name: post_name_change,
    post_content: post_content_change
  }
  console.log(updatedPost)

  storagePost.value.put(updatedPost).then((response: any) => {
    console.log(response);
    fetchData();
    post_content_change = ""
    post_name_change = ""
    //location.reload();
  }).catch((err: any) => {
    console.log(err);
  });

  return;
}

let comment_content_change=""

const updateComment = (comment: any) => {

  console.log(comment.comment_content)

  const updatedPost = {
    ...comment,
    comment_content: comment_content_change
  }
  console.log(updatedPost)

  storageComment.value.put(updatedPost).then((response: any) => {
    console.log(response);
    fetchData();
    comment_content_change = ""
    //location.reload();
  }).catch((err: any) => {
    console.log(err);
  });

  return;
}

const deleteDocument = (post: any) => {

  console.log(post._id)

  storagePost.value.remove(post._id, post._rev).then((result: any) => {
    console.log(result);
    fetchData();
    //location.reload();
  }).catch((err: any) => {
    console.log(err);
  });
  return;
}

const deleteComment = (comment: any) => {

  console.log(comment._id)

  storageComment.value.remove(comment._id, comment._rev).then((result: any) => {
    console.log(result);
    fetchData();
    //location.reload();
  }).catch((err: any) => {
    console.log(err);
  });
  return;
}

onMounted(() => {
  console.log('=> Composant initialisé');
  initDatabase()
  fetchData()
  //addDocument()
  //updateDocument()
  //deleteDocument()
});

const getComment = (postId: any) => {

  // que les commentaires de commentDatas qui ont le post ID
  return commentsData.value.filter((doc) => doc.post_id === postId);
}

const openReadmeInEditor = () => fetch('/__open-in-editor?file=README.md')
</script>

<template>
  <h1>Add Post</h1>
  <input type="text" id="postName" name="postName" v-model="post_name" placeholder="post name" />
  <input type="text" id="postContent" name="postContent" v-model="post_content" placeholder="post content" />
  <button @click="addDocument()">Add Document</button>
  <h1>Posts</h1>

  <article v-for="post in postsData" v-bind:key="(post as any).id">
    <div>

      <h2>{{ post.post_name }}</h2>
      <input type="text" id="postNameChange" name="postNameChange" v-model="post_name_change"
        placeholder="change post name" />
      <button @click="updateDocument(post)">Update Post</button>

      <p>{{ post.post_content }}</p>
      <input type="text" id="postContentChange" name="postContentChange" v-model="post_content_change"
        placeholder="change post content" />
      <button @click="deleteDocument(post)">Delete Post</button>

      <hr></hr>
      <button @click="addLike(post)">Like</button>
      <p>{{ post.post_like }}</p>

      <div>
        <input type="text" id="commentContent" name="commentContent" v-model="comment_content" placeholder="comment" />
        <button @click="addComment(post)">Comment Post</button>
        <h3>Comments</h3>
        <article v-for="comment in getComment(post._id)" v-bind:key="(comment as any).id">
          <div>
            <p>{{ comment.comment_content }}</p>
            <input type="text" id="commentContentChange" name="commentContentChange" v-model="comment_content_change"
              placeholder="change comment content" />
            <button @click="updateComment(comment)">Update Comment</button>
            <button @click="deleteComment(comment)">Delete Comment</button>
          </div>
          <p></p>

        </article>
      </div>
    </div>
  </article>

  <h1>Replicate</h1>
  <button @click="() => {
    PouchDB.sync('post', post)
    PouchDB.sync('comment', comment)
  }">Sync</button>
  <button @click="dataCreateIndex()">Create Index</button>


</template>
