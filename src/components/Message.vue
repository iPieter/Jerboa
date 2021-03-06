<template>
  <div
    :class="incremental ? 'message-container-inc pt-2' : 'message-container'"
    @mouseover="hovered = true"
    @mouseleave="hovered = false"
  >
    <img
      class="avatar"
      :src="base + 'files?f=' + sender.profile_image"
      v-if="!incremental"
    />
    <div class="message">
      <div class="header" v-if="!incremental">
        <span class="font-weight-bold">{{ sender.first_name }}</span>
        <span class="font-weight-light text-muted ml-2 d-sm-none">{{
          smalltime
        }}</span>
        <span class="font-weight-light text-muted ml-2 d-none d-sm-inline">{{
          time
        }}</span>
      </div>

      <message-input
        v-if="editMode"
        :send="handleSend"
        :paste="handlePaste"
        :escape="() => (editMode = false)"
        :messageProp="messages[messages.length - 1].message"
        :emojis="emojis"
      ></message-input>
      <div v-else class="content mt-0">
        <div
          v-if="
            messages[messages.length - 1].message_type == 'TEXT_MESSAGE' ||
              messages[messages.length - 1].message_type ==
                'TEXT_MESSAGE_UPDATE'
          "
        >
          <vue-markdown
            :emoji="true"
            :prerender="preMessageRender"
            :postrender="postMessageRender"
            class="content-msg"
            :source="messages[messages.length - 1].message"
          ></vue-markdown>
          <span class="font-weight-light text-muted" v-if="edited">
            <i>(edited)</i>
          </span>
          <span>
            <i
              class="fas fa-pen icon"
              v-if="hovered"
              @click="editMode = true"
            ></i>
            <i class="fas fa-share icon" v-if="hovered" @click="share"></i>
          </span>
        </div>
        <div v-else-if="filesAreImages()">
          <vue-markdown
            v-if="messages[messages.length - 1].message.message.length > 0"
            :emoji="true"
            :prerender="preMessageRender"
            :postrender="postMessageRender"
            class="content-msg"
            :source="messages[messages.length - 1].message.message"
          ></vue-markdown>
          <div
            class="preview mb-2"
            v-for="(file, index) in messages[messages.length - 1].message.files"
            v-bind:key="index"
          >
            <a href="#" v-on:click="showFullImage(index)">
              <img
                class="mx-1 my-2"
                :src="base + 'files?f=' + file.file"
                v-bind:key="index"
              />
            </a>
          </div>
        </div>
        <div
          v-else-if="
            messages[messages.length - 1].message_type == 'SHARE_MESSAGE'
          "
        >
          <blockquote class="mb-0">
            <img
              class="d-inline-block small-avatar"
              :src="
                base +
                  'files?f=' +
                  $root.$data.users[
                    messages[messages.length - 1].message.source.sender
                  ].profile_image
              "
            />

            <b class>{{
              $root.$data.users[
                messages[messages.length - 1].message.source.sender
              ].first_name
            }}</b>
            said
            <br />
            <vue-markdown
              :emoji="true"
              :prerender="preMessageRender"
              :postrender="postMessageRender"
              class="content-msg"
              :source="messages[messages.length - 1].message.source.message"
            ></vue-markdown>
          </blockquote>
          <vue-markdown
            :emoji="true"
            :prerender="preMessageRender"
            :postrender="postMessageRender"
            class="content-msg"
            :source="messages[messages.length - 1].message.message"
          ></vue-markdown>
        </div>
        <div v-else>
          <vue-markdown
            :emoji="true"
            :prerender="preMessageRender"
            :postrender="postMessageRender"
            class="content-msg"
            :source="messages[messages.length - 1].message.message"
          ></vue-markdown>
          <b-card class="m-2 files-card">
            <b-card-title
              class="m-3"
              v-if="messages[messages.length - 1].message.files.length != 1"
              >{{ sender.first_name }} shared
              {{ messages[messages.length - 1].message.files.length }}
              files</b-card-title
            >
            <b-card-title class="m-3" v-else
              >{{ sender.first_name }} shared a file</b-card-title
            >
            <table class="card-table table">
              <tbody role="rowgroup">
                <tr
                  role="row"
                  v-for="(file, index) in messages[messages.length - 1].message
                    .files"
                  :key="index"
                >
                  <td class="icon-cell">
                    <code
                      v-if="file.type == 'text/plain'"
                      class="fas fa-file-code"
                    ></code>
                    <code
                      v-else-if="file.type == 'application/pdf'"
                      class="fas fa-file-pdf"
                    ></code>
                    <code
                      v-else-if="file.type == 'application/zip'"
                      class="fas fa-file-archive"
                    ></code>
                    <code
                      v-else-if="file.type == 'application/vnd.ms-powerpoint'"
                      class="fas fa-file-powerpoint"
                    ></code>
                    <code
                      v-else-if="file.type.startsWith('image/')"
                      class="fas fa-file-image"
                    ></code>
                    <code v-else class="fas fa-file"></code>
                  </td>
                  <td>
                    <code class>{{ file.full_name }}</code>
                  </td>
                  <td>
                    <span class>{{ formatBytes(file.size) }}</span>
                  </td>
                  <td>
                    <small class="text-muted">{{ file.type }}</small>
                  </td>
                  <td class="text-right">
                    <b-button-group>
                      <b-button
                        variant="outline-primary btn-sm"
                        v-bind:href="base + 'files?f=' + file.file"
                      >
                        <i class="fas fa-file-download"></i>
                      </b-button>
                      <b-button
                        variant="outline-primary btn-sm"
                        v-if="file.type == 'application/pdf'"
                        v-on:click="
                          togglePreview(base + 'files?show=1&f=' + file.file)
                        "
                      >
                        <i class="fas fa-search"></i>
                      </b-button>
                    </b-button-group>
                  </td>
                </tr>
              </tbody>
            </table>
            <b-card-text class="m-2 small text-muted">
              Download all files as a
              <a href>zip archive</a>.
            </b-card-text>
          </b-card>
        </div>
      </div>
    </div>
    <div
      class="modal-image"
      v-if="modal_id >= 0"
      v-on:click.self="showFullImage(-1)"
    >
      <div class="closing" v-on:click="showFullImage(-1)">
        <i class="far fa-times-circle"></i>
      </div>
      <div class="metadata">
        <div class>
          <img
            class="d-inline-block small-avatar"
            :src="base + 'files?f=' + sender.profile_image"
          />
          <b class="mr-1">{{ sender.first_name }}</b
          >shared this image
        </div>

        <div class="previews mb-2">
          <a
            v-for="(file, index) in messages[messages.length - 1].message.files"
            href="#"
            class="preview"
            v-on:click="showFullImage(index)"
            v-bind:key="index"
          >
            <img
              class="mx-1 my-1"
              :class="index == modal_id ? 'active' : ''"
              :src="base + 'files?f=' + file.file"
            />
          </a>
        </div>
      </div>
      <img
        class="image"
        :src="
          base +
            'files?f=' +
            messages[messages.length - 1].message.files[modal_id].file
        "
      />
    </div>
    <b-modal
      id="modal-prevent-closing"
      ref="modal"
      title="Share message"
      @show="resetModal"
      @hidden="resetModal"
      @ok="handleOk"
      v-if="
        messages[messages.length - 1].message_type == 'TEXT_MESSAGE' ||
          messages[messages.length - 1].message_type == 'TEXT_MESSAGE_UPDATE'
      "
    >
      <blockquote>
        <img
          class="d-inline-block small-avatar"
          :src="base + 'files?f=' + sender.profile_image"
        />
        <b class>{{ sender.first_name }}</b> said
        <br />
        <vue-markdown
          :emoji="true"
          :postrender="postMessageRender"
          class="content-msg"
          :source="messages[messages.length - 1].message"
        ></vue-markdown>
      </blockquote>
      <form ref="form" @submit.stop.prevent="handleSubmit">
        <b-form-group label="Your reply" label-for="message-input">
          <MessageInput
            :send="handleSubmit"
            :paste="handlePaste"
            :escape="() => this.$refs.modal.hide()"
            :emojis="$root.$data.custom_emojis"
            id="message-input"
            ref="replyInput"
          ></MessageInput>
        </b-form-group>
      </form>
    </b-modal>
  </div>
</template>

<script>
import Vue from "vue";
import VueMarkdown from "vue-markdown";
import moment from "moment";
import MessageInput from "./MessageInput";

Vue.use(VueMarkdown);
moment.locale("nl-be");

export default {
  name: "message",
  data() {
    return {
      base: "/",
      editMode: false,
      edited: false,
      messages: [],
      hovered: false,
      modal_id: -1,
      scroll_position: 0,
    };
  },
  components: {
    VueMarkdown,
    MessageInput,
  },
  props: {
    id: {
      type: Number,
      required: true,
    },
    incremental: {
      type: Boolean,
    },
    messagesProp: {
      type: Array,
      required: true,
    },
    emojis: {
      type: Array,
    },
    token: {
      required: true,
    },
    sender: {
      required: true,
    },
  },
  beforeMount() {
    this.base = process.env.VUE_APP_SERVER_BASE;
    this.messages.splice(0, this.messages.length);
    this.messagesProp.forEach((m) => this.messages.push(m));
    this.edited = this.messages.length > 1;
  },
  watch: {
    messagesProp: {
      handler() {
        this.edited = true;
        this.messages.splice(0, this.messages.length);
        this.messagesProp.forEach((m) => this.messages.push(m));
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    togglePreview(url) {
      if (this.$parent.$parent.$data.file_preview == "") {
        this.$parent.$parent.$data.file_preview = url;
      } else {
        this.$parent.$parent.$data.file_preview = "";
      }
    },
    toggleEdit() {
      this.editMode = true;
    },
    showFullImage(id) {
      console.log(window.scrollY);
      if (id >= 0) this.scroll_position = window.scrollY;

      this.modal_id = id;
      window.scrollTo(window.scrollX, this.scroll_position);
    },
    filesAreImages() {
      /*
      Returns true if the message contains files and those
      are all images, false otherwise.
      */
      if (
        this.messages[this.messages.length - 1].message_type != "FILES_MESSAGE"
      ) {
        return false;
      }

      let filesAreImages = true;

      for (let f in this.messages[this.messages.length - 1].message.files) {
        console.log("files");
        filesAreImages =
          filesAreImages &&
          this.messages[this.messages.length - 1].message.files[
            f
          ].type.startsWith("image/");
      }

      console.log(filesAreImages);
      return filesAreImages;
    },
    updateMessage(msg) {
      this.edited = true;
      this.messages.push(msg);
    },

    handleSend(content) {
      var msg = {
        message_type: "TEXT_MESSAGE_UPDATE",
        sender: this.token,
        channel: "1",
        message: content,
        sent_time: new Date(),
        signature: "na",
        previous_message: this.id,
        nonce: "",
      };

      this.editMode = false;
      this.$root.$data.socket.emit("msg", JSON.stringify(msg));
    },
    resetModal() {
      this.sharingMessage = "";
    },
    handleOk(bvModalEvt) {
      // Prevent modal from closing
      bvModalEvt.preventDefault();
      // Trigger submit handler
      this.handleSubmit(this.$refs.replyInput.getMessage());
    },
    handleSubmit(message) {
      var msg = {
        message_type: "SHARE_MESSAGE",
        sender: this.token,
        channel: "1",
        message: {
          source: this.messages[this.messages.length - 1],
          message: message,
        },
        nonce: "",
        sent_time: new Date(),
        signature: "na",
      };

      this.$root.$data.socket.emit("msg", JSON.stringify(msg));

      // Hide the modal manually
      this.$nextTick(() => {
        this.$refs.modal.hide();
      });
    },
    share() {
      this.$refs.modal.show();
    },
    handlePaste() {},
    preMessageRender(data) {
      if (data.startsWith(">>>")) {
        return "<blockquote>" + data.replace(/^(>>>)/, "") + "</blockquote>";
      }
      return data;
    },
    postMessageRender(htmlData) {
      let re = /:([A-z0-9-]+(:{2}\S+)?):/g;

      let data = htmlData.replace(
        new RegExp(re),
        "<img class='emoji' src='" +
          process.env.VUE_APP_SERVER_BASE +
          "emoji/$1'/ alt=':$1:'/>"
      );

      //check if there is something left, otherwise insert h1 tag
      let text = htmlData
        .substring(3, htmlData.length - 5)
        .replace(new RegExp(re), "")
        .trim();
      if (!text) data = "<h1>" + data + "</h1>";

      return data;
    },
    formatBytes(bytes) {
      if (bytes == 0) {
        return "0 B";
      }

      var formatArr = ["B", "kB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];
      var tmptotal;
      var f = 0;

      //round function rounds up, not down. eg. round(30.5,0) will round to 31
      var len = String(Math.round(bytes)).length;

      if (len < 4) {
        tmptotal = bytes;
      } else if (len >= 4 && len < 7) {
        tmptotal = bytes / 1024;
        ++f;
      } else if (len >= 7 && len < 10) {
        tmptotal = bytes / 1024 / 1024;
        f += 2;
      } else if (len >= 10 && len < 13) {
        tmptotal = bytes / 1024 / 1024 / 1024;
        f += 3;
      } else if (len >= 13 && len < 16) {
        tmptotal = bytes / 1024 / 1024 / 1024 / 1024;
        f += 4;
      } else if (len >= 16) {
        tmptotal = bytes / 1024 / 1024 / 1024 / 1024 / 1024;
        f += 5;
      } else {
        tmptotal = bytes;
      }

      tmptotal = tmptotal.toFixed(1);

      //handle if too great of a value and format is passed in
      if (f > 8) {
        return bytes + " B";
      }

      var new_format_out = formatArr[f];
      //round it off it is already in bits
      if (new_format_out == "B") {
        tmptotal = new Number(tmptotal).toFixed(0);
      }

      //strips all trailing zeroes, note the escaped period
      tmptotal = tmptotal.toString().replace(/\.0\$/, "");

      return tmptotal + " " + new_format_out;
    },
  },
  computed: {
    time: function() {
      return moment(
        new Date(this.messages[this.messages.length - 1].sent_time * 1000)
      ).format("LLL");
    },

    smalltime: function() {
      return moment(
        new Date(this.messages[this.messages.length - 1].sent_time * 1000)
      ).format("HH:mm");
    },
  },
};
</script>

<style lang="scss">
@import "../style.scss";

.small-avatar {
  border-radius: 3pt;
  margin-right: 4px;
  height: 1.3em;
  width: 1.3em;
  margin-top: -5px;
}

.emoji {
  display: inline-block;
  height: 2.75ex;
  padding: 0;
  background: transparent;
  margin: 0;
  box-shadow: unset;
  margin-top: -0.5ex;
}

.message-container,
.message-container-inc {
  line-height: 1;
  padding-right: 50px;

  .avatar {
    height: 26pt;
    line-height: 26pt;
    width: 26pt;
    transform: translate(0, 16px);
    position: relative;
    vertical-align: top;
    margin-right: 5pt;
    border-radius: 3pt;
    margin-top: -8px;
  }

  .message {
    display: inline-block;
    width: 100%;
    margin-left: 40px;
    margin-right: -50px;

    p {
      margin-bottom: 0.8ex;
    }

    h1 {
      margin-top: 0.5ex;
    }

    .content {
      line-height: 1.5;
    }
    .header {
      display: block;
    }
    .content {
      display: block;
      width: 100%;
    }

    .content-msg {
      display: inline-block;
      width: 90%;
    }

    .content-ctrl {
      display: inline-block;
      width: 10%;
    }

    .content .preview {
      display: inline-block;

      img {
        max-height: 20ex;
        max-width: 90%;
        orientation: landscape;
        padding: 0.5rem;
      }
    }

    .content .icon {
      width: 16px;
      margin-left: 5px;
      margin-top: 0;
      background: transparent;
      padding: 0;
      opacity: 0.5;
      box-shadow: none;
      color: #3498db;
      &:hover {
        opacity: 1;
      }
    }

    pre {
      padding-left: 4px;
      margin-left: 5px;
      margin-top: 10px;
      border-left: 4px solid #3498db;
      border-radius: 2px;
    }

    blockquote {
      padding-left: 4px;
      margin-left: 5px;
      margin-top: 10px;
      border-left: 4px solid #bdc3c7;
      border-radius: 2px;
    }

    .emoji {
      background: transparent;
      box-shadow: none;
      border: none;
    }
  }
}

.modal-dialog {
  blockquote {
    padding-left: 4px;
    margin-left: 5px;
    margin-top: 10px;
    border-left: 4px solid #bdc3c7;
    border-radius: 2px;
  }
}

.message-container {
  border-top: 1px solid #eff0f1;
  margin-bottom: -2pt;

  .message {
    margin-top: -18px;
  }
}

.message-container-inc {
  margin-top: -10px;
  margin-left: 0;
}

.files-card {
  max-width: 700px;
  .card-table {
    width: 100%;
    border-bottom: 1px solid #dee2e6;

    .icon-cell {
      width: 16px;
      padding-right: 0;
    }

    code {
      color: #34495e;
    }
  }
}

.closing {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10000;
  color: rgba(255, 255, 255, 0.8);
  margin-left: 20px;
  margin-top: 20px;
  font-size: 24px;

  &:hover {
    color: rgba(255, 255, 255, 1);
  }
}

.modal-image {
  position: fixed;
  margin-top: 60px;
  width: 100vw;
  z-index: 9999;
  bottom: 0;
  top: 0;
  right: 0;
  left: 0;
  @include alpha-attribute(
    "background-color",
    rgba(darken($jerboa_color4, 30%), 0.7),
    black
  );
  display: flex;
  align-items: center;
  -webkit-backdrop-filter: blur(5px);
  backdrop-filter: blur(5px);
  padding-bottom: 114px;

  .image {
    max-height: calc(90vh - 114px);
    max-width: 90vw;
    display: inline-block;
    margin: auto;
    orientation: landscape;
    padding: 0.5rem;
  }

  .metadata {
    position: absolute;
    z-index: 10000;
    bottom: 0;
    width: 100vw;
    border-top: 1px solid rgba(255, 255, 255, 0.3);
    width: 100%;
    color: rgba(255, 255, 255, 0.9);
    background-color: rgba(0, 0, 0, 0.15);
    padding: 10px;

    .previews {
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: row;

      .preview {
        display: inline-block;
        opacity: 0.9;
        z-index: 10001;

        img {
          width: 60px;
          border: 4px solid #fff;
          border-radius: 5px;
          box-shadow: 0 0.2rem 1.2rem rgba(255, 255, 255, 0.1);
        }
        .active {
          opacity: 1;
          border: 4px solid #3498db;
          border-radius: 5px;
        }
      }
    }
  }
}

@media screen and (prefers-color-scheme: dark) {
  .message,
  .modal-image {
    img {
      background: #868686;
      box-shadow: 0 0.2rem 1.2rem rgba(255, 255, 255, 0.1);
    }
  }
}
@media screen and (prefers-color-scheme: light) {
  .message,
  .modal-image {
    img {
      background: #fff;
      box-shadow: 0 0.2rem 1.2rem rgba(0, 0, 0, 0.1);
    }
  }
}
</style>
