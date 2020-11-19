<template>
  <div v-if="isCommentsVisible" class="central_window comments">
    <div class="centered_div">
      <div class="central_text">
        <h4>Comments 🗣️</h4>
        <div id="container_comments">
          <div
            v-for="(element, key) in listComments"
            v-bind:key="key"
            id="container_message"
          >
            <li v-if="element">
              <b>{{ element.username }}</b> said : (on {{ element.date }})<br />
              <div class="content_message">{{ element.message }}</div>
            </li>
          </div>
        </div>
        <br />
        <form @submit.prevent="createComment">
          <textarea
            v-model="username"
            rows="1"
            cols="50"
            placeholder="Enter a username..."
            class="form_field"
          ></textarea>
          <br />
          <textarea
            v-model="message"
            rows="4"
            cols="50"
            placeholder="Add a message..."
            class="form_field"
          ></textarea>
          <br />
          <button @click="getComments()" class="button_menu small_button">
            Refresh</button
          >&nbsp;&nbsp;
          <button type="submit" class="button_menu small_button">Send</button>
        </form>
      </div>
      <button class="button_menu" @click="comeBackToMenu()">Menu</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { onMounted, ref } from "vue";
export default {
  emits: {
    comebacktomenu: null,
  },
  props: {
    isCommentsVisible: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, { emit }) {
    let message = ref(null);
    let username = ref(null);
    let listComments = ref({});

    let comeBackToMenu = function () {
      emit("comebacktomenu");
    };

    let getComments = function () {
      axios
        .get("/comments.php")
        .then(function (response) {
          let newListComments = {};

          let result = response.data.split(";");

          // for each message
          result.forEach((element) => {
            element = element.split(",");

            if (element.length > 1) {
              let foundId = element[0];
              let foundDate = element[1];
              let foundUsername = element[2];
              let foundMessage = element.slice(3, element.length);

              foundMessage = foundMessage.join();

              // date formatting
              var t = foundDate.split(/[- :]/);

              foundDate = new Date(
                Date.UTC(t[0], t[1] - 1, t[2], t[3], t[4], t[5])
              );

              newListComments[foundId] = {
                date: foundDate.toLocaleString(),
                username: foundUsername,
                message: foundMessage,
              };
            }
          });

          // sort by date
          newListComments = Object.values(newListComments).sort((a, b) => {
            return new Date(a.date) - new Date(b.date);
          });

          listComments.value = newListComments;
        })
        .catch(function (error) {
          console.log(error);
        });
    };

    let createComment = function () {
      // preliminary checks
      if (!username.value || !message.value) {
        return;
      }

      // date formatting
      let dateCreatiomMessage = new Date()
        .toISOString()
        .slice(0, 19)
        .replace("T", " ");

      let formData = new FormData();
      formData.append("username", username.value);
      formData.append("message", message.value);
      formData.append("date", dateCreatiomMessage);

      axios({
        method: "post",
        url: "/comments.php",
        data: formData,
        config: { headers: { "Content-Type": "multipart/form-data" } },
      })
        .then(function () {
          //handle success
          getComments();
          message.value = "";
        })
        .catch(function (response) {
          //handle error
          console.log(response);
        });
    };

    onMounted(() => {
      getComments();
    });

    return {
      message,
      username,
      listComments,
      comeBackToMenu,
      createComment,
      getComments,
    };
  },
};
</script>

<style>
#container_comments {
  height: 80%;
  max-height: 350px;
  border: 2px solid white;
  align-items: center;
  justify-content: center;
  margin: auto;
  border-radius: 5%;
  overflow: scroll;
  -ms-overflow-style: none; /* IE and Edge */
  scrollbar-width: none; /* Firefox */
}

/* Hide scrollbar for Chrome, Safari and Opera */
#container_comments::-webkit-scrollbar {
  display: none;
}

#container_message li {
  list-style: none;
  margin: 0;
  padding: 0;
  text-align: start;
  margin: 10px;
  white-space: pre-wrap;
}

.comments {
  align-items: unset;
}

.comments .centered_div{
  grid-template-rows: 85% 15%;
}

.comments .central_text{
  grid-template-rows: 65px auto 20px 150px;
  display: grid;
}

.form_field {
  min-width: 0;
  width: calc(100% - 10px);
}

.small_button {
  height: 30px;
}

.content_message {
  margin: 0 10px;
}
</style>