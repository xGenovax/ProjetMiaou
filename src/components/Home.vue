<template>
    <v-card flat>
  <v-card-title primary-title>
    <h4>Connexion</h4>
  </v-card-title>
  <v-form ref="form" v-model="valid" lazy-validation >
        <v-text-field
        prepend-icon="person"
      v-model="pseudo"
      :rules="nameRules"
      :counter="10"
      label="Quel est votre nom ?"
      required
    ></v-text-field>
    <v-btn
  :disabled="!valid"
  @click="submit"
>
  Valider
</v-btn>
  </v-form>
</v-card >
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      nameRules: [
        v => !!v || 'Nom requis',
        v => v.length <= 10 || 'Le nom doit faire moins de 10 caractères'
      ],
      pseudo: "",
      valid: true,
      chooseRoom: false,
      roomName: null,
      muted: false,
      waitingToast: null
    };
  },
  mounted: function() {

  },
  methods: {
    submit(){
      if(this.valid)
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
