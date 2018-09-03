<template>
  <div class="hello">
    <v-toolbar dark color="primary">
      <v-toolbar-side-icon></v-toolbar-side-icon>

      <v-toolbar-title class="white--text">{{title}}</v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn icon>
        <v-icon>search</v-icon>
      </v-btn>

      <v-btn icon @click="addSalon">
        <v-icon>add</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>refresh</v-icon>
      </v-btn>

      <v-btn icon>
        <v-icon>more_vert</v-icon>
      </v-btn>
  </v-toolbar>
    <!-- <h2>Chat experience</h2>
    <v-list subheader>
  <v-subheader>Recent chat</v-subheader>
  <v-list-tile
    v-for="item in items"
    :key="item.title"
    avatar
    @click=""
  >
    <v-list-tile-avatar>
      <img :src="item.avatar">
    </v-list-tile-avatar>

    <v-list-tile-content>
      <v-list-tile-title v-html="item.title"></v-list-tile-title>
    </v-list-tile-content>

    <v-list-tile-action>
      <v-icon :color="item.active ? 'teal' : 'grey'">chat_bubble</v-icon>
    </v-list-tile-action>
  </v-list-tile>
</v-list> -->

<v-divider></v-divider>

<v-list subheader>
  <v-subheader>Utilisateurs connectés</v-subheader>

  <v-list-tile
    v-for="(item,key) in utilisateurs"
    :key="key"
    avatar
    @click="choixCible(item,key)"
  >
    <v-list-tile-avatar>
      <v-icon>face</v-icon>
    </v-list-tile-avatar>

    <v-list-tile-content>
      <v-list-tile-title v-html="item.pseudo"></v-list-tile-title>
    </v-list-tile-content>
  </v-list-tile>
</v-list>

<v-divider></v-divider>

<v-list subheader>
  <v-subheader>Salons</v-subheader>

  <v-list-tile
    v-for="(item,key) in salons"
    :key="key"
    avatar
    @click="choixSalon(key)"
  >
    <v-list-tile-avatar>
      <v-icon>face</v-icon>
    </v-list-tile-avatar>

    <v-list-tile-content>
      <v-list-tile-title v-html="getTitre(item)"></v-list-tile-title>
    </v-list-tile-content>
  </v-list-tile>
</v-list>


<v-list subheader v-if="selectedSalon">
  <v-subheader>Chat</v-subheader>

  <v-list-tile
    v-for="(item,key) in salons[selectedSalon].messages"
    :key="key"
    avatar
  >
    <v-list-tile-avatar>
      <v-icon>face</v-icon>
    </v-list-tile-avatar>

    <v-list-tile-content>
      <v-list-tile-title v-html="item.pseudo+' : '+item.message"></v-list-tile-title>
    </v-list-tile-content>
  </v-list-tile>
</v-list>
    <!-- <ul>
      <li v-for="task in utilisateurs" @click="choixCible(task)">
        {{ task.pseudo }}
      </li>
    </ul> -->
    <!-- <input v-model="nomCible.pseudo" type="text"/> -->
    <!-- <button @click="click">OK</button>
    <button @click="click2">OK2</button> -->

    <v-text-field
    prepend-icon="msg"
  v-model="message"
  :rules="nameRules"
  :counter="100"
  label="Message"
></v-text-field>
<v-btn
@click="submit"
>
Envoyer
</v-btn>

            <div> Votre camera: <v-divider></v-divider><video ref="videoLocal" id="video" width="640" height="480" autoplay muted></video></div>
            <div>{{nomCible.pseudo}}<v-divider></v-divider><video ref="videoExt" id="video" width="640" height="480" autoplay muted></video></div>
  </div>
</template>

<script>
import Firebase from 'firebase'


var Peer = require('simple-peer');
var wrtc = require('wrtc');






export default {
  name: 'HelloWorld',
  data () {
    return {
      db:null,
      utilisateursRef:null,
      salonsRef:null,
      utilisateurs:[],
      title: 'Miaou3',
      nomCible:{
          pseudo:null
      },
      salons:[],
      selectedSalon:'',
      selectedSalonRef:null,
      db:null,
      booksRef2:null,
      booksRef:null,
      pseudo:"",
      onlineRef:null,
      peertab: {},
      message:"",
      nameRules: [
        v => !!v || 'Name is required',
        v => v.length <= 10 || 'Name must be less than 10 characters'
      ]
    }
  },
  mounted() {

    var config = {
      apiKey: "AIzaSyA6rhTTjM9Tyu23pe1Ug6FK8s6C7vdRgSA",
      authDomain: "miaou-87713.firebaseapp.com",
      databaseURL: "https://miaou-87713.firebaseio.com",
      projectId: "miaou-87713",
      storageBucket: "miaou-87713.appspot.com",
      messagingSenderId: "88588984979"
    };

    var app = !firebase.apps.length ? firebase.initializeApp(config) : firebase.app();
    this.db = app.database();
    this.utilisateursRef = this.db.ref('utilisateurs');
    this.utilisateursRef.on('value', (snapshot)=> {
      const val = snapshot.val();
      this.utilisateurs = val;
    })

    this.salonsRef = this.db.ref('salons');
    this.salonsRef.on('value', (snapshot)=> {
      const val = snapshot.val();
      this.salons = val;
    })
    //booksRef.push({test:"test"})

    this.videoLocal = this.$refs.videoLocal;
    this.videoExt = this.$refs.videoExt;

    if(!this.$route.params.userPseudo){
      this.$router.push({ name: 'Home'});
    }

    this.pseudo = this.$route.params.userPseudo;
    if(this.pseudo){
      console.log(this.pseudo);
      //this.utilisateurs = db.ref('utilisateurs');
      this.onlineRef = this.db.ref('utilisateurs').push({pseudo:this.pseudo});
      this.onlineRef.onDisconnect().remove(() => {
      this.deconnectSalon();
    });
      console.log("onlineRef ",this.onlineRef);
      //this.utilisateurs.onDisconnect().removeValue();
      this.afficherSelfCamera();


      //if(this.pseudo === "jo")
        console.log("books/"+this.onlineRef.key)
        this.db.ref('books').child(this.onlineRef.key).on('child_added', (snapshot)=>{
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

        this.db.ref('books').child(this.onlineRef.key).on('child_changed', (snapshot)=>{
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
     this.db.ref((isInitiator?'books2/'+target:'books/'+target['.key']+'/'+this.onlineRef.key)).set({
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
     this.db.ref('books').child(this.onlineRef.key).off();
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
choixCible(nomCible,key){
  this.nomCible = {
    'pseudo':nomCible.pseudo,
    '.key':key
  };
  this.click();
},
choixSalon(salon){
  if(salon != this.selectedSalon){
    this.deconnectSalon();
    this.selectedSalon = salon;
    this.selectedSalonRef = this.db.ref('salons').child(salon);
    this.selectedSalonRef.child("membres").child(this.onlineRef.key).set(true);
    this.selectedSalonRef.child("membres").child(this.onlineRef.key).onDisconnect().remove();
  }
},
addSalon(event, nom = "monSalon"){
  console.log("Ajouter Salon ",nom);
  this.db.ref('salons').push({nom});;
},
getTitre(salon){
  if(salon.membres){
    return salon.nom + ' ('+Object.keys(salon.membres).length+')'
  }else{
    return salon.nom
  }
},
deconnectSalon(){
  if(this.selectedSalon){
    this.db.ref('salons').child(this.selectedSalon).child("membres").child(this.onlineRef.key).remove();
  }
},
submit(event){
  console.log({pseudo:this.pseudo,message:this.message})
  this.selectedSalonRef.child("messages").push({pseudo:this.pseudo,message:this.message});
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
  this.db.ref('books2/'+this.onlineRef.key).off();
  //this.peer1 = null;
  // this.createPeer(true, () => {
  //   console.log("calling", inputCallee)
  // })

  this.createPeer(false,this.nomCible, () => {
    this.db.ref('books2/'+this.onlineRef.key).on('value', (snapshot)=>{

      // this.peer1.signal(snapshot.val().data)
      if(snapshot.val())
      {
          console.log("voyeur: recu ",snapshot.val())
          this.peer2.signal(snapshot.val().data)//… or just process the message
      }
    });
  })



//Declencheur
  this.db.ref('books/'+this.nomCible['.key']+'/'+this.onlineRef.key).set({
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
