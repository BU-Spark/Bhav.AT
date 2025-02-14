<template>
  <main>
    <section class="jumbotron">
      <div class="container">
        <h3>Project Chat</h3>
        <div class="my-3 p-3 bg-white rounded shadow-sm scroll" v-chat-scroll>
          <div class="media text pt-3">
            <div>
              <div
                class="media-body pb-3 mb-0 lh-125"
                v-for="chat in chats"
                :key="chat.id"
              >
                <span>{{ chat.value }}</span>
                <span class="text-muted"
                  >{{ chat.sender }},
                  {{ makeTimeReadable(chat.timestamp) }}</span
                >
                <a
                  v-if="user.uname == chat.sender"
                  class="btn text-right text-muted"
                  @click.prevent="
                    delComment(chat.value, chat.sender, chat.timestamp)
                  "
                  >Delete</a
                >
              </div>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-12">
            <form>
              <div class="input-group add-on">
                <input
                  type="text"
                  class="form-control"
                  id="newMessage"
                  placeholder="Send a chat..."
                  v-model="newMessage"
                  @input="checkInput()"
                />
                <div class="input-group-btn">
                  <button
                    :disabled="disableSend"
                    class="btn btn-primary"
                    @click.prevent="addComment()"
                  >
                    Send
                  </button>
                </div>
              </div>
            </form>
          </div>
        </div>
      </div>
    </section>

    <hr class="featurette-divider" />

    <footer class="container">
      <p class="float-right"><a href="#">Back to top</a></p>
      <p>
        © 2020 BU Spark! ·
        <a>
          <router-link :to="{ name: 'privacy' }">Privacy</router-link>
        </a>
      </p>
    </footer>
  </main>
</template>
<script>
import moment from "moment";
import { db, functions } from "@/firebase/init.js";
export default {
  computed: {
    user() {
      return this.$store.state.user;
    }
  },
  data() {
    return {
      route: this.$route.path,
      chats: [],
      chatBoxID: null,
      newMessage: null,
      disableSend: null,
      members: null,
      name: this.$route.params.name
    };
  },
  methods: {
    async addComment() {
      await functions.httpsCallable("addProjectChat")({
        chatID: this.chatBoxID,
        message: {
          value: this.newMessage,
          sender: this.user.uname,
          timestamp: Date.now()
        }
      });
      this.newMessage = null;
    },
    async delComment(value, sender, timestamp) {
      await functions.httpsCallable("deleteProjectChat")({
        chatID: this.chatBoxID,
        message: {
          value: value,
          sender: sender,
          timestamp: timestamp
        }
      });
    },
    async checkInput() {
      if (this.newMessage == null || this.newMessage == "") {
        this.disableSend = true;
      } else {
        this.disableSend = false;
      }
    },
    makeTimeReadable(timestamp) {
      return moment(timestamp).format("lll");
    }
  },
  async created() {
    let findChat = await db
      .collection("projectchats")
      .where("chatBoxID", "array-contains", this.route)
      .get();
    let usercheck = await db
      .collection("projects")
      .where("name", "==", this.name)
      .get();
    this.members = usercheck.docs[0].data().members;
    let check = this.members.find(item => item == this.user.uname);
    if (check != undefined) {
      if (findChat.empty) {
        await functions.httpsCallable("createProjectChatBox")({
          chatBoxID: this.route
        });
        location.reload();
      } else {
        this.chatBoxID = findChat.docs[0].id;
      }
      const msgref = db.collection("projectchats").doc(this.chatBoxID);
      msgref.onSnapshot(docSnapshot => {
        this.chats = docSnapshot.data().chats;
      });
      this.disableSend = true;
    } else {
      this.$router.push({
        name: "project",
        params: {
          name: this.$route.params.name
        }
      });
    }
  }
};
</script>

<style scoped>
.container {
  padding-top: 40px;
  padding-bottom: 40px;
}

.material-icons.green {
  color: green;
}

.material-icons.red {
  color: red;
}

.availability {
  padding-top: 6px;
  padding-left: 3px;
}

.section-head {
  padding-left: 8px;
}

.content {
  padding-left: 12px;
}

.green {
  color: green;
}

.red {
  color: red;
}

.scroll {
  position: relative;
  z-index: 10;
  max-height: 400px;
  overflow-y: scroll;
}

.add-on .input-group-btn > .btn {
  border-left-width: 0;
  left: -2px;
  -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
}

.add-on .form-control:focus {
  box-shadow: none;
  -webkit-box-shadow: none;
  border-color: #cccccc;
}

.text-muted {
  display: block;
  font-size: 0.8rem;
}

.material-icons.md-18 {
  font-size: 18px;
}

.material-icons.md-24 {
  font-size: 24px;
}

.material-icons.md-36 {
  font-size: 36px;
}

.material-icons.md-48 {
  font-size: 48px;
}
</style>
