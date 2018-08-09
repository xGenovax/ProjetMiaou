<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Chat experience {{pseudo}}</h2>
    <ul>
      <li v-for="task in utilisateurs" @click="choixCible(task)">
        {{ task.pseudo }}
      </li>
    </ul>
    <input v-model="nomCible.pseudo" type="text"/>
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
      msg: 'Miaou3',
      nom: null,
      nomCible:{
          pseudo:null
      },
      db:null,
      booksRef2:null,
      booksRef:null,
      pseudo:"null",
      onlineRef:null,
      peertab: {}
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
    if(this.pseudo){
      console.log(this.pseudo);
      this.nom = this.pseudo;
      //this.utilisateurs = db.ref('utilisateurs');
      this.onlineRef = db.ref('utilisateurs').push({pseudo:this.pseudo});
      this.onlineRef.onDisconnect().remove();
      console.log("onlineRef ",this.onlineRef);
      //this.utilisateurs.onDisconnect().removeValue();
      this.afficherSelfCamera();


      //if(this.pseudo === "jo")
        console.log("books/"+this.onlineRef.key)
        db.ref('books').child(this.onlineRef.key).on('child_added', (snapshot)=>{
          console.log("on child_added ",snapshot.val())

              if(!this.peertab[snapshot.val().caller]){
                if(snapshot.val()){
                  console.log("peer recepteur ",snapshot.val())
                  this.createPeer(true,snapshot.val().caller, () => {
                    if(snapshot.val().data){
                      this.peertab[snapshot.val().caller].signal(snapshot.val().data)//… or just process the message
                    } //… and process the message
                  },snapshot.val().caller)
                }

              }else{
                this.peertab[snapshot.val().caller].signal(snapshot.val().data)//… or just process the message
              }




            // // this.peer1.signal(snapshot.val().data)
            // if (!this.peer1) { //… create a peer if doesn’t exist…
            //   console.log("peer recepteur ")
            //   this.createPeer(true,snapshot.val().caller, () => {
            //     if(snapshot.val().data){
            //     this.peer1.signal(snapshot.val().data)} //… and process the message
            //   })
            // } else {
            //   this.peer1.signal(snapshot.val().data)//… or just process the message
            // }

        });

        db.ref('books').child(this.onlineRef.key).on('child_changed', (snapshot)=>{
          console.log("on child_changed ",snapshot.val())

              if(!this.peertab[snapshot.val().caller]){
                if(snapshot.val()){
                  console.log("peer recepteur ",snapshot.val())
                  this.createPeer(true,snapshot.val().caller, () => {
                    if(snapshot.val().data){
                      this.peertab[snapshot.val().caller].signal(snapshot.val().data)//… or just process the message
                    } //… and process the message
                  },snapshot.val().caller)
                }

              }else{
                this.peertab[snapshot.val().caller].signal(snapshot.val().data)//… or just process the message
              }

        });
    }


},
methods: {
  afficherSelfCamera(){
    navigator.mediaDevices.getUserMedia({ audio: false, video: true }).then(stream => {
      //videoLocal montre la camera de l'utilisateur
      this.videoLocal.srcObject = stream;
      this.videoLocal.play();
    })
  },
  createPeer(isInitiator, target, callback,key) {
    console.log("createPeer ",target)
 return navigator.mediaDevices.getUserMedia({ audio: false, video: true }).then(stream => {
   //videoLocal montre la camera de l'utilisateur
   //this.videoLocal.srcObject = stream;

   //J'envoie le stream video
   //console.log("this.peer1 ",this.peer1)
   var peer = isInitiator ? new Peer({ initiator: isInitiator, wrtc: wrtc,stream: stream }) : new Peer({ wrtc: wrtc });
   console.log(isInitiator?"Je suis initiator, j'envoie le stream":"je recois le stream")
   peer.on("signal", data => { //when a message is ready to be sent…
     //Envoyer data à l'autre utilisateur
     db.ref((isInitiator?'books2/'+target:'books/'+target['.key']+'/'+this.onlineRef.key)).set({
      caller: this.onlineRef.key,
      data: data
    })
   })

   peer.on('stream', (stream)=>{
     console.log("STREAM")
      // got remote video stream, now let’s show it in a video tag
     this.videoExt.srcObject = stream;
     this.videoExt.play();
   });

   peer.on("close", () => { //call is ended
     console.log("close")
     /* … some DOM related code … */
     db.ref('books').child(this.onlineRef.key).off();
     peer = null;
     this.videoExt.srcObject = "";
     this.videoLocal.srcObject = "";
   })

  if(isInitiator){
    //this.peer1 = peer;
    this.peertab[key] = peer;
  }else{
    this.peer2 = peer;
  }
  //this.peer1.signal(target.data)
  callback()
 })

},
choixCible(nomCible){
  console.log(nomCible)
  this.nomCible = {
    'pseudo':nomCible.pseudo,
    '.key':nomCible['.key']
  };
},
click2(){
  // console.log("click2")
  // db.ref('books/'+this.nom).on('value', (snapshot)=>{
  //   console.log("recu ",snapshot.val())
  //   // this.peer1.signal(snapshot.val().data)
  //   if (!this.peer1) { //… create a peer if doesn’t exist…
  //     this.createPeer(true, () => {
  //       this.peer1.signal(snapshot.val().data) //… and process the message
  //     })
  //   } else {
  //     this.peer1.signal(snapshot.val().data)//… or just process the message
  //   }
  // });



},
click(){
  console.log("CLICK")
  this.peer2 = null;
  db.ref('books2/'+this.onlineRef.key).off();
  //this.peer1 = null;
  // this.createPeer(true, () => {
  //   console.log("calling", inputCallee)
  // })

  this.createPeer(false,this.nomCible, () => {
    db.ref('books2/'+this.onlineRef.key).on('value', (snapshot)=>{

      // this.peer1.signal(snapshot.val().data)
      if(snapshot.val())
      {
          console.log("voyeur: recu ",snapshot.val())
          this.peer2.signal(snapshot.val().data)//… or just process the message
      }
    });
  })



//Declencheur
  db.ref('books/'+this.nomCible['.key']+'/'+this.onlineRef.key).set({
   caller: this.onlineRef.key
 })

  // this.onlineRef.child('watchers').on('value', (snapshot)=>{
  //   console.log("recu ",snapshot.val())
  //   if (!this.peer1) { //… create a peer if doesn’t exist…
  //     this.createPeer(true, () => {
  //       this.peer1.signal(snapshot.val().data) //… and process the message
  //     })
  //   } else {
  //     this.peer1.signal(snapshot.val().data)//… or just process the message
  //   }
  // });
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
