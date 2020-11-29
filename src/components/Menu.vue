<template>
  <div v-if="!isMenuOpen && isMenuVisible" @click="toggleMenu()" id="menu_icon">
    <span>&nbsp;</span>
    <span>&nbsp;</span>
    <span>&nbsp;</span>
  </div>
  <div v-if="isMenuOpen && isMenuVisible" class="central_window">
    <div class="centered_div height_limited">
      <button @click="startGame()" class="button_menu">Play</button>
      <button @click="setInfoVisible()" class="button_menu">Info</button>
      <button @click="setCommentsVisible()" class="button_menu">
        Comments
      </button>
      <button @click="setContactVisible()" class="button_menu">Contact</button>
      <button @click="toggleMenu()" class="button_menu">Home</button>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  emits: [
    "setinfovisible",
    "setcontactvisible",
    "startgame",
    "setcommentsvisible",
  ],
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
      emit("setinfovisible");
    };

    let setContactVisible = function () {
      emit("setcontactvisible");
    };

    let startGame = function () {
      emit("startgame");
    };

    let setCommentsVisible = function () {
      emit("setcommentsvisible");
    };

    return {
      isMenuOpen,
      toggleMenu,
      setInfoVisible,
      setContactVisible,
      setCommentsVisible,
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
  margin: 10px;
  height: 24px;
  width: 24px;
  color: black;
  background-color: rgb(255, 255, 255, 0.4);
  border-radius: 50%;
  border: 2px solid black;
  cursor: pointer;
  display: inline-block;
}

#menu_icon span {
  width: 10px;
  background-color: #000;
  height: 2px;
  display: block;
  margin: 2px 7px;
}

#menu_icon span:first-child {
  margin-top: 7px;
}
#menu_icon span:last-child {
  margin-bottom: 0px;
}

.central_window {
  position: absolute;
  top: 20px;
  left: 20px;
  height: calc(100% - 40px);
  width: calc(100% - 40px);
  min-width: 0;
  background: rgb(0, 0, 0, 0.7);
  border-radius: 5%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.button_menu {
  height: 40px;
  width: 80px;
  margin: auto;
  cursor: pointer;
  background-color: black;
  color: white;
  border: 1px solid white;
  box-shadow: 0px 0px 6px white;
  align-items: center;
  justify-content: center;
}
</style>