<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Chat experience {{pseudo}}</h2>
    <ul>
      <li v-for="task in utilisateurs" @click="choixCible(task.pseudo)">
        {{ task.pseudo }}
      </li>
    </ul>
    <input v-model="nom" type="text"/>
    <input v-model="nomCible" type="text"/>
    <button @click="click">OK</button>
    <button @click="click2">OK2</button>
            <div><video ref="videoLocal" id="video" width="640" height="480" autoplay muted></video></div>
            <div><video ref="videoExt" id="video" width="640" height="480" autoplay muted></video></div>
  </div>
</template>

<script>
import Firebase from 'firebase'


var Peer = require('simple-peer');
var wrtc = require('wrtc');


var config = {
  apiKey: "AIzaSyA6rhTTjM9Tyu23pe1Ug6FK8s6C7vdRgSA",
  authDomain: "miaou-87713.firebaseapp.com",
  databaseURL: "https://miaou-87713.firebaseio.com",
  projectId: "miaou-87713",
  storageBucket: "miaou-87713.appspot.com",
  messagingSenderId: "88588984979"
};
var app = firebase.initializeApp(config);
var db = app.database()
let utilisateursRef = db.ref('utilisateurs');


export default {
  name: 'HelloWorld',
  firebase: {
    utilisateurs: utilisateursRef
  },
  data () {
    return {
      msg: 'Miaou2',
      nom:"titi",
      nomCible:"toto",
      db:null,
      booksRef2:null,
      booksRef:null,
      pseudo:"null"
    }
  },
  mounted() {

    //booksRef.push({test:"test"})

    this.videoLocal = this.$refs.videoLocal;
    this.videoExt = this.$refs.videoExt;

    if(!this.$route.params.userPseudo){
      this.$router.push({ name: 'Home'});
    }

    this.pseudo = this.$route.params.userPseudo;
    console.log(this.pseudo);
    this.nom = this.pseudo;
    //this.utilisateurs = db.ref('utilisateurs');
    db.ref('utilisateurs').push({pseudo:this.pseudo});
    //this.utilisateurs.onDisconnect().removeValue();
    this.afficherSelfCamera();

},
methods: {
  afficherSelfCamera(){
    navigator.mediaDevices.getUserMedia({ audio: false, video: true }).then(stream => {
      //videoLocal montre la camera de l'utilisateur
      this.videoLocal.srcObject = stream;
      this.videoLocal.play();
    })
  },
  createPeer(isInitiator, callback) {
 navigator.mediaDevices.getUserMedia({ audio: false, video: true }).then(stream => {
   //videoLocal montre la camera de l'utilisateur
   //this.videoLocal.srcObject = stream;

   //J'envoie le stream video
   this.peer1 = new Peer({ initiator: isInitiator, wrtc: wrtc,stream: stream });

   this.peer1.on("signal", data => { //when a message is ready to be sent…
     console.log("sending", JSON.stringify(data))
     //Envoyer data à l'autre utilisateur
     db.ref('books/'+this.nomCible).set({
      caller: this.nom,
      data: data
    })
   })

   this.peer1.on('stream', (stream)=>{
     console.log("STREAM")
      // got remote video stream, now let’s show it in a video tag
     this.videoExt.srcObject = stream;
     this.videoExt.play();
   });

   this.peer1.on("close", () => { //call is ended
     /* … some DOM related code … */
     this.peer1 = null;
     this.videoExt.srcObject = "";
     this.videoLocal.srcObject = "";
   })

  callback()

 })

},
choixCible(nomCible){
  this.nomCible = nomCible;
},
click2(){
  console.log("click2")
  this.createPeer(true, () => {
  console.log("calling")
})
},
click(){
  console.log("CLICK")
  this.peer1 = null;
  // this.createPeer(true, () => {
  //   console.log("calling", inputCallee)
  // })


  db.ref('books/'+this.nom).on('value', (snapshot)=>{
    console.log("recu ",snapshot.val())
    // this.peer1.signal(snapshot.val().data)
    if (!this.peer1) { //… create a peer if doesn’t exist…
      this.createPeer(false, () => {
        this.peer1.signal(snapshot.val().data) //… and process the message
      })
    } else {
      this.peer1.signal(snapshot.val().data)//… or just process the message
    }
  });
  //
  // if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
  //     navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
  //         //videoLocal montre la camera de l'utilisateur
  //         this.videoLocal.srcObject =stream;
  //
  //         //J'envoie le stream video
  //         this.peer1 = new Peer({ initiator: true, wrtc: wrtc,stream: stream });
  //
  //         this.peer2 = new Peer({ wrtc: wrtc });
  //
  //         booksRef.on('child_added', (snapshot, prevChildKey)=>{
  //           console.log("ADDED ",snapshot.val())
  //           this.peer1.signal(snapshot.val().data)
  //         });
  //
  //         this.peer1.on('signal', data => {
  //           //when a message is ready to be sent…
  //           console.log("peer1 signal",data);
  //
  //           //Envoyer data à l'autre utilisateur
  //           booksRef2.push({
  //            caller: this.nom,
  //            data: data
  //          })
  //           //this.peer2.signal(data);
  //         })
  //
  //         // this.peer2.on('signal', data => {
  //         //   console.log("peer2 signal",data);
  //         //   this.peer1.signal(data)
  //         // })
  //
  //         this.peer1.on('stream', (stream)=>{
  //           console.log("STREAM")
  //            // got remote video stream, now let’s show it in a video tag
  //           this.videoExt.srcObject = stream;
  //           this.videoExt.play();
  //         });

          //this.videoLocal.play();
      //});
//   }
 }
}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
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
