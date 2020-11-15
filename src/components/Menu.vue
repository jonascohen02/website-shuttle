<template>
  <div v-if="!isMenuOpen && isMenuVisible" @click="toggleMenu()" id="menu_icon">
    &#9776;
  </div>
  <div v-if="isMenuOpen && isMenuVisible" class="central_window">
    <div class="centered_div">
      <button @click="startGame()" class="button_menu">Play</button>
      <button @click="setInfoVisible()" class="button_menu">Info</button>
      <button @click="setContactVisible()" class="button_menu">Contact</button>
      <button @click="toggleMenu()" class="button_menu">Home</button>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  emits: ["set-info-visible", "set-contact-visible", "start-game"],
  props: {
    isMenuVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let isMenuOpen = ref(false);

    let toggleMenu = function () {
      isMenuOpen.value = !isMenuOpen.value;
    };

    let setInfoVisible = function () {
      emit("set-info-visible");
    };

    let setContactVisible = function () {
      emit("set-contact-visible");
    };

    let startGame = function () {
      emit("start-game");
    };

    return {
      isMenuOpen,
      toggleMenu,
      setInfoVisible,
      setContactVisible,
      startGame,
    };
  },
};
</script>

<style>
#menu,
#menu_icon {
  position: absolute;
  top: 0;
  left: 0;
}

#menu_icon {
  font-size: 25px;
  margin: 10px;
  color: white;
  cursor: pointer;
}

.central_window {
  position: absolute;
  top: 20px;
  left: 20px;
  height: calc(100% - 40px);
  width: calc(100% - 40px);
  background: rgb(0, 0, 0, 0.7);
  border-radius: 5%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.button_menu {
  height: 40px;
  width: 80px;
  margin: 0 auto;
  cursor: pointer;
  background-color: black;
  color: white;
  border: 1px solid white;
  box-shadow: 0px 0px 6px white;
  align-items: center;
  justify-content: center;
}
</style>