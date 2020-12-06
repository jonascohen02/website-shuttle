<template>
  <div v-if="isContactVisible" class="central_window">
    <div class="centered_div">
      <div class="central_text">
        <br />
        <h4>Contact 📬 </h4>
        <br />
        <br />
        <br />
        <form @submit.prevent="sendMail">
          <textarea
            v-model="object"
            rows="1"
            cols="50"
            placeholder="Enter a subject. (Optionnal)"
            class="form_field"
          ></textarea>
          <br />
          <br />
          <textarea
            v-model="message"
            rows="4"
            cols="50"
            placeholder="Send me a private message. Let me a way to answer you 😉"
            class="form_field"
          ></textarea>
          <br />
          <br />
          <br />
          <button type="submit" class="button_menu small_button">Send</button>
        </form>
      </div>
      <button class="button_menu" @click="comeBackToMenu()">Menu</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref } from "vue";

export default {
  emits: ["comebacktomenu"],
  props: {
    isContactVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let message = ref("");
    let object = ref("");
    let comeBackToMenu = function () {
      emit("comebacktomenu");
    };

    let sendMail = function () {
      // preliminary checks
      if (!message.value) {
        return;
      }

      let formData = new FormData();
      formData.append("object", object.value || "No subject.");
      formData.append("message", message.value);

      axios({
        method: "post",
        url: "http://thomas.simmer.free.fr/mail.php",
        data: formData,
        config: { headers: { "Content-Type": "multipart/form-data" } },
      })
        .then(function (response) {
          //handle success
          console.log(response);
          message.value = "Your message was sent successfully !";
          setTimeout(() => {
            message.value = "";
          }, 3000);
        })
        .catch(function (response) {
          //handle error
          console.log(response);
        });
    };

    return {
      comeBackToMenu,
      sendMail,
      message,
      object,
    };
  },
};
</script>

<style>
</style>