<template>
  <div class="text-edit">
    <div v-if="showEmojiHelper && emojiPredictions.length > 0" class="col-11 emoji-predictor">
      <div
        v-for="(emoji, index) in emojiPredictions"
        v-bind:key="index"
        class="emoji-prediction"
        v-bind:class="{ 'emoji-selected': index === currentSelectedEmoji }"
      >
        {{ emoji.name }}
        <img :src="base_url + emoji.imageUrl" />
      </div>
    </div>
    <textarea
      ref="textareaInput"
      placeholder="Message"
      autofocus
      @keydown.tab.exact="handleTab"
      @keydown.tab.shift.exact="handleReverseTab"
      @keydown.right="handleTab"
      @keydown.left="handleReverseTab"
      @keydown.escape.exact="handleEscape"
      @keydown.tab.exact.prevent
      @keydown.enter.exact.prevent
      @keyup.enter.exact="handleEnter"
      @keyup.up.exact="handleKeyUp"
      @keydown.enter.shift.exact="newline"
      @keypress="updateEmojiShower"
      @focusout="handleEscape"
      v-model="message"
      v-bind:rows="rows"
      v-on:paste="paste"
    />
  </div>
</template>
<script>
import Vue from "vue";
import VueFuse from "vue-fuse";
Vue.use(VueFuse);

export default {
  name: "message-input",
  props: {
    send: {
      type: Function,
      required: true
    },
    paste: {
      type: Function,
      required: true
    },
    emojis: {
      type: Array,
      required: true
    },
    escape: {
      type: Function
    },
    keyup: {
      type: Function
    },
    typingCallback: {
      type: Function
    },
    messageProp: {}
  },
  data() {
    return {
      message: "",
      rows: 1,
      showEmojiHelper: false,
      emojiPredictions: [],
      currentSelectedEmoji: 0,
      base_url: process.env.VUE_APP_SERVER_BASE
    };
  },
  mounted() {
    this.$refs.textareaInput.focus();
    if (this.messageProp) this.message = this.messageProp;
  },
  methods: {
    newline() {
      this.message = this.message + "\n";
      this.rows++;
    },
    handleEscape() {
      this.showEmojiHelper = false;
      if (this.escape) this.escape();
    },
    handleKeyUp(event) {
      if (this.keyup) this.keyup(event);
    },
    handleTab(event) {
      if (this.showEmojiHelper) {
        event.preventDefault();
        this.currentSelectedEmoji =
          (this.currentSelectedEmoji + 1) % this.emojiPredictions.length;
      }
    },
    handleReverseTab(event) {
      if (this.showEmojiHelper) {
        event.preventDefault();
        this.currentSelectedEmoji--;
        if (this.currentSelectedEmoji < 0)
          this.currentSelectedEmoji = this.emojiPredictions.length - 1;
      }
    },
    handleEnter() {
      if (this.showEmojiHelper) {
        var name = this.emojiPredictions[this.currentSelectedEmoji].name;
        var textarea = this.$refs.textareaInput;
        var lastColon = this.message.lastIndexOf(":", textarea.selectionStart);
        if (lastColon >= 0) {
          this.message =
            this.message.substring(0, lastColon) + ":" + name + ":" + " ";
          this.showEmojiHelper = false;
          this.currentSelectedEmoji = 0;
        }
      } else {
        this.send(this.message);
        this.message = "";
        this.rows = 1;
      }
    },
    getMessage() {
      return this.message;
    },
    resetMessage() {
      this.message = "";
    },
    setMessage(message) {
      this.message = message;
    },
    updateEmojiShower(event) {
      var textarea = this.$refs.textareaInput;

      if (this.typingCallback) {
        this.typingCallback();
      }

      var lastColon = this.message.lastIndexOf(":", textarea.selectionStart);

      if (lastColon >= 0) {
        var lastWord = this.message.substring(
          lastColon,
          textarea.selectionStart + 1
        );
        lastWord += event.key;
        this.showEmojiHelper = !/\s/g.test(lastWord);
        if (this.showEmojiHelper) {
          var searchOptions = {
            keys: ["name"],
            threshold: 0.2
          };
          this.$search(lastWord.substring(1), this.emojis, searchOptions)
            .then(result => {
              this.emojiPredictions = result.slice(
                0,
                Math.min(result.length, 10)
              );
            })
            .catch(err => {
              console.log(err);
            });
        }
      } else if (lastColon == -1) {
        this.showEmojiHelper = false;
      }
    }
  }
};
</script>
<style scoped>
.text-edit textarea {
  display: block;
  width: 100%;
  height: calc(1.5em + 0.75rem + 2px);
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
  resize: none;
}
.text-edit {
  flex: 1;
}
.emoji-predictor {
  border: 1px solid #ced4da;
  border-radius: 5px;
  padding: 3px;
  background: #f9f9f9;
  z-index: 99999;
  position: absolute;
  bottom: 60px;
  color: #777;
}

.emoji-prediction {
  display: inline-block;
  padding: 3px;
  margin: 2px;
}
.emoji-prediction img {
  width: 1em;
}
.emoji-selected {
  border-radius: 5px;
  background: #007bff0e;
  border: 1px solid #007bff;
  color: #666;
  padding: 2px;
}

@media screen and (prefers-color-scheme: dark) {
  .text-edit textarea {
    background-color: rgb(38, 35, 36);
    border-color: rgb(52, 51, 51);
    color: #bdc3c7;
  }
}
</style>
