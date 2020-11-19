<template>
  <div v-if="isInfoVisible" class="central_window">
    <div class="centered_div">
      <div class="central_text">
        <br />
        <br />
        <h4>Info 🚀</h4>
        <br />
        <br />
        <br />
        The aim of this website was to learn
        <a href="https://v3.vuejs.org/" target="_blank">Vue3</a> and
        <a href="https://threejs.org/" target="_blank">Three.js</a>.
        <br />
        <br />
        Find the code of this project
        <a
          href="https://github.com/thomassimmer/Website-Shuttle"
          target="_blank"
          >here</a
        >.
        <br />
        <br />
        Author :
        <a
          href="https://www.linkedin.com/in/thomas-simmer-b104b2158/"
          target="_blank"
          >Thomas Simmer</a
        >
        <br />
        <br />
        You want to support my development ?
        <a href="https://www.paypal.com/paypalme/ThomasSimmer" target="_blank">Why not ?</a>
        <br />
        <br />
        This website was visited {{ visits }} times.
      </div>
      <button class="button_menu" @click="comeBackToMenu()">Menu</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
export default {
  emits: ["comebacktomenu"],
  props: {
    isInfoVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let visits = ref(0);

    let comeBackToMenu = function () {
      emit("comebacktomenu");
    };

    // Website visits counter
    onMounted(() => {
      var xhr = new XMLHttpRequest();
      xhr.open(
        "GET",
        "https://api.countapi.xyz/hit/thomas.simmer.free.fr/visits"
      );
      xhr.responseType = "json";
      xhr.onload = function () {
        visits.value = this.response.value.toString();
      };
      xhr.send();
    });

    return {
      visits,
      comeBackToMenu,
    };
  },
};
</script>

<style>
a {
  color: cadetblue;
  text-underline-position: under;
}
</style>