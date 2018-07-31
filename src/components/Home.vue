<template>
  <div class="hello">
    Quel est votre nom ?
    <input type="text" v-model="pseudo"/>
    <button @click="click" >Valider</button>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      pseudo: null,
      chooseRoom: false,
      roomName: null,
      muted: false,
      waitingToast: null
    };
  },
  mounted: function() {

  },
  methods: {
    click(){

      if(this.pseudo)
        {
          console.log("click");
          this.$router.push({ name: 'HelloWorld', params: { userPseudo: this.pseudo }});
        }
    }
  }
};
</script>

<style>

</style>
