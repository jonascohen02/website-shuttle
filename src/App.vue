<template>
  <div v-if="appIsLoading" id="loader">
    <div class="centered_div height_limited">
      <div
        v-for="(element, key) in listObjectToLoad"
        v-bind:key="key"
        class="elementLoading"
      >
        <div v-if="!element">
          Loading {{ key }} ...&nbsp;
          <div class="spinner"></div>
        </div>
        <div v-else>
          {{ key }} loaded.&nbsp;
          <div class="loaded"></div>
        </div>
      </div>
    </div>
  </div>
  <Menu
    :isMenuVisible="isMenuVisible"
    @setinfovisible="setInfoVisible"
    @setcontactvisible="setContactVisible"
    @setcommentsvisible="setCommentsVisible"
    @startgame="startGame"
  />
  <Info :isInfoVisible="isInfoVisible" @comebacktomenu="setMenuVisible" />
  <Comments
    :isCommentsVisible="isCommentsVisible"
    @comebacktomenu="setMenuVisible"
  />
  <Contact
    :isContactVisible="isContactVisible"
    @comebacktomenu="setMenuVisible"
  />
  <Background
    :isGameEnable="isGameEnable"
    @backgroundhasloaded="updateSetup"
    @stopgame="stopGame"
  />
</template>

<script>
import Background from "@/components/Background";
import Menu from "@/components/Menu";
import Info from "@/components/Info";
import Contact from "@/components/Contact";
import Comments from "@/components/Comments";

import { ref } from "vue";

export default {
  name: "App",
  components: {
    Background,
    Menu,
    Info,
    Contact,
    Comments,
  },
  setup() {
    let isMenuVisible = ref(true);
    let isInfoVisible = ref(false);
    let isContactVisible = ref(false);
    let isCommentsVisible = ref(false);
    let isGameEnable = ref(false);
    let listObjectToLoad = ref({
      Sun: false,
      Earth: false,
      Shuttle: false,
      Moon: false,
      Stars: false,
    });
    let appIsLoading = ref(true);

    let sortObject = function (unordered) {
      var ordered = {};
      Object.keys(unordered)
        .sort()
        .forEach(function (key) {
          ordered[key] = unordered[key];
        });
      return ordered;
    };

    let setAllUnvisible = function () {
      isMenuVisible.value = false;
      isInfoVisible.value = false;
      isContactVisible.value = false;
      isCommentsVisible.value = false;
    };

    let setMenuVisible = function () {
      setAllUnvisible();
      isMenuVisible.value = true;
    };

    let setInfoVisible = function () {
      setAllUnvisible();
      isInfoVisible.value = true;
    };

    let setContactVisible = function () {
      setAllUnvisible();
      isContactVisible.value = true;
    };

    let setCommentsVisible = function () {
      setAllUnvisible();
      isCommentsVisible.value = true;
    };

    let startGame = function () {
      setAllUnvisible();
      isGameEnable.value = true;
    };

    let stopGame = function () {
      isGameEnable.value = false;
      isMenuVisible.value = true;
    };

    /**
     * set obj has loaded in the list of object to load and check if everyone
     * is loaded before setting appIsLoading to false
     * @param {String} obj ex: "Sun"
     */
    let updateSetup = async function (obj) {
      listObjectToLoad.value[obj] = true;
      let isReady = true;
      for (let key in listObjectToLoad.value) {
        if (!listObjectToLoad.value[key]) isReady = false;
      }
      if (isReady) {
        setTimeout(() => {
          appIsLoading.value = false;
        }, 300);
      }
    };

    listObjectToLoad.value = sortObject(listObjectToLoad.value);

    return {
      isMenuVisible,
      isInfoVisible,
      isContactVisible,
      isCommentsVisible,
      isGameEnable,
      appIsLoading,
      listObjectToLoad,
      setInfoVisible,
      setContactVisible,
      setCommentsVisible,
      setMenuVisible,
      startGame,
      updateSetup,
      stopGame,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
html,
body,
#app,
#loader {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

/* Block scroll in ios */
body,
html {
  position: fixed;
}

.central_text {
  color: white;
  min-width: 0;
  height: 100%;
}

@media (max-width: 470px) {
  .central_text {
    font-size: 3vw;
  }
}

#loader {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: black;
  color: white;
  display: flex;
  box-sizing: border-box;
  align-items: center;
  justify-content: center;
}

.spinner {
  border: 3px solid #f3f3f3;
  border-radius: 50%;
  border-top: 3px solid #3498db;
  width: 12px;
  height: 12px;
  -webkit-animation: spin 1s linear infinite; /* Safari */
  animation: spin 1s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.elementLoading {
  margin: auto;
  width: 80%;
  max-width: 300px;
}

.elementLoading div {
  display: flex;
  align-items: center;
  justify-content: center;
}

.loaded {
  background-image: url("/textures/done_icon.png");
  width: 17px;
  height: 17px;
  background-size: 100%;
}

.centered_div {
  height: 100%;
  display: grid;
  width: calc(100% - 40px);
  max-width: 450px;
}

.height_limited {
  max-height: 300px;
}
</style>
