<template>
  <div>
    <menu-lateral :empresa="empresa"></menu-lateral>

    <q-page class="q-px-lg q-py-md">
      <router-view></router-view>
    </q-page>
  </div>
</template>

<script>
import MenuLateral from "../../shared/MenuLateral.vue";

export default {
  data() {
    return {
      drawer: false,
      miniState: false,
      empresa: {},
    };
  },

  components: {
    MenuLateral,
  },

  created() {
    window.setTimeout(() => {
      this.empresa = this.$store.state.usuario.empresa;
      this.$http
        .get(`empresa/${this.empresa.id}`)
        .then((res) => res.json())
        .then((empresa) => {
          this.empresa = empresa;
          let url = `${this.$http.options.root}/empresa/${this.empresa.id}/images/logo`;
          this.empresa.imgUrl = url;
        }),
        (err) => {
          console.log(err);
        };
    }, 1000);
  },
};
</script>

<style scoped>
</style>