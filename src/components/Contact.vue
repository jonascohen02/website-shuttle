<template>
  <div v-if="isContactVisible" class="central_window">
    <div class="centered_div">
      <div class="central_text">Contact is visible</div>
      <button class="button_menu" @click="comeBackToMenu()">Menu</button>
    </div>
  </div>
</template>

<script>
export default {
  emits: ["come-back-to-menu"],
  props: {
    isContactVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let comeBackToMenu = function () {
      console.log(0);
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