<template>
  <div class="container">
    <h2 class="text-center"><i class="fas fa-question mr-2"></i>質問投稿</h2>
    <Form :isAnswer="false"></Form>
    <div class="text-right">
      <button @click="createQuestion()" class="btn btn-primary mb-3 mb-md-5"><i class="fas fa-reply mr-2"></i>質問投稿</button>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
import axios from "axios";
import Form from "./Form.vue";

export default {
  components: {
    Form,
  },
  props: {
    tagList: Boolean,
  },
  setup(props, context) {
    const router = useRouter();
    const store = useStore();

    const data = reactive({
      isPosting: false,
    });

    function beginPost() {
      data.isPosting = true;
    }

    function endPost() {
      data.isPosting = false;
    }

    onMounted(() => {
      context.emit("showTagList", props.tagList);
      store.state.questionDetails.content = "";
    });

    async function createQuestion() {
      if (store.state.questionDetails.title == "" || store.state.questionDetails.content == "") {
        store.commit("setAlert", {
          flag: {
            showSuccessAlert: false,
            showErrorAlert: true,
            showWarningAlert: false,
          },
          message: {
            error: "タイトル、内容を入力してから投稿してください。",
          },
        });
      } else {
        if (!data.isPosting) {
          beginPost();
          await axios
            .post(
              `${process.env.VUE_APP_CONNECT_BACKEND_URL}/questions/create`,
              {
                question: {
                  title: store.state.questionDetails.title,
                  content: store.state.questionDetails.content,
                  anonymous: store.state.questionDetails.anonymous,
                  solved: 0,
                  tag_ids: store.state.selected_tags_id,
                },
              },
              { withCredentials: true }
            )
            .then((response) => {
              if (response.data.posted) {
                store.commit("resetQuestionDetails");
                store.commit("resetTagId");
                store.commit("setAlert", {
                  flag: {
                    showSuccessAlert: true,
                    showErrorAlert: false,
                  },
                  message: {
                    success: "投稿に成功しました",
                  },
                });
                router.push({
                  name: "home",
                });
              } else {
                store.commit("setAlert", {
                  flag: {
                    showSuccessAlert: false,
                    showErrorAlert: true,
                  },
                  message: {
                    error: "投稿に失敗しました。",
                  },
                });
              }
            })
            .catch((e) => {
              alert(e);
            });
          endPost();
        }
      }
    }
    return {
      data,
      beginPost,
      endPost,
      createQuestion,
    };
  },
};
</script>
