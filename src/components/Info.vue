<template>
  <div v-if="isInfoVisible" class="central_window">
    <div class="centered_div">
      <div class="central_text">Info is visible</div>
      <button class="button_menu" @click="comeBackToMenu()">Menu</button>
    </div>
  </div>
</template>

<script>
export default {
  emits: ["come-back-to-menu"],
  props: {
    isInfoVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let comeBackToMenu = function () {
      emit("come-back-to-menu");
    };
    return {
      comeBackToMenu,
    };
  },
};
</script>

<style>
</style>