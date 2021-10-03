<template>
  <div class="sc-message-list" ref="scrollList" :style="{backgroundColor: colors.messageList.bg}" @scroll="handleScroll">
    <Message v-for="(message, idx) in messages" :message="message" :user="profile(message.author)" :key="idx" :colors="colors" :messageStyling="messageStyling" :showEdition="showEdition" :showDeletion="showDeletion" @remove="$emit('remove', message)">
      <template v-slot:user-avatar="scopedProps">
        <slot name="user-avatar" :user="scopedProps.user" :message="scopedProps.message">
        </slot>
      </template>
      <template v-slot:text-message-body="scopedProps">
        <slot name="text-message-body" :message="scopedProps.message" :messageText="scopedProps.messageText" :messageColors="scopedProps.messageColors" :me="scopedProps.me">
        </slot>
      </template>
      <template v-slot:system-message-body="scopedProps">
        <slot name="system-message-body" :message="scopedProps.message">
        </slot>
      </template>
      <template v-slot:text-message-toolbox="scopedProps">
        <slot name="text-message-toolbox" :message="scopedProps.message" :me="scopedProps.me">
        </slot>
      </template>
    </Message>
    <Message
      v-show="showTypingIndicator !== ''"
      :message="{author: showTypingIndicator, type: 'typing'}"
      :user="profile(showTypingIndicator)"
      :colors="colors"
      :messageStyling="messageStyling"
      :showEdition="showEdition"
      :showDeletion="showDeletion" />

      
    
    <UserInput
      v-if="!showUserList"
      :showEmoji="showEmoji"
      :showPhotos="showPhotos"
      :showMap="showMap"
      :showCredits="showCredits"
      :showHelp="showHelp"
      :showSend="showSend"
      :onSubmit="onSubmit"
      :suggestions="getSuggestions()"
      :showFile="showFile"
      :placeholder="placeholder"
      @onType="$emit('onType')"
      @edit="$emit('edit', $event)"
      :colors="colors"
      :mapSetting="mapSetting"
      :onlyButton="true"
      :messages="messages" />
  </div>
</template>
<script>
import Message from './Message.vue'
import UserInput from './UserInput.vue'
import chatIcon from './assets/chat-icon.svg'

export default {
  components: {
    Message,
    UserInput
  },
  props: {
    participants: {
      type: Array,
      required: true
    },
    messages: {
      type: Array,
      required: true
    },
    showTypingIndicator: {
      type: String,
      required: true
    },
    colors: {
      type: Object,
      required: true
    },
    mapSetting: {
      type: Object,
      required: true
    },
    alwaysScrollToBottom: {
      type: Boolean,
      required: true
    },
    messageStyling: {
      type: Boolean,
     required: true
    },
    showEdition: {
      type: Boolean,
      required: true
    },
    showEmoji: {
      type: Boolean,
      default: false
    },
    showPhotos: {
      type: Boolean,
      default: false
    },
    showMap: {
      type: Boolean,
      default: false
    },
    showCredits: {
      type: Boolean,
      default: false
    },
    showHelp: {
      type: Boolean,
      default: false
    },
    showSend: {
      type: Boolean,
      default: false
    },
    onSubmit: {
      type: Function,
      required: true
    },
    showFile: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: 'Write a message...'
    },
  },
  methods: {
    _scrollDown () {
      //this.$refs.scrollList.scrollTop = this.$refs.scrollList.scrollHeight
      this.$refs.scrollList.scrollTop = this.$refs.scrollList.scrollTop+20
      
      /*
      setTimeout(() => {
        this.$refs.scrollList.scrollTop = this.$refs.scrollList.scrollTop+20
      }, 100);
      */
    },
    handleScroll (e) {
        if (e.target.scrollTop === 0) {
            this.$emit('scrollToTop')
        }
    },
    shouldScrollToBottom() {
      return this.alwaysScrollToBottom || (this.$refs.scrollList.scrollTop > this.$refs.scrollList.scrollHeight - 600)
    },
    profile(author) {
      const profile = this.participants.find(profile => profile.id === author)

      // A profile may not be found for system messages or messages by 'me'
      return profile || {imageUrl: '', name: ''}
    },
    getSuggestions(){
      return this.messages.length > 0 ? this.messages[this.messages.length - 1].suggestions : []
    }
  },
  computed: {
    defaultChatIcon() {
      return chatIcon
    }
  },
  mounted () {
    this.$nextTick(this._scrollDown())
  },
  updated () {
    if (this.shouldScrollToBottom())
      this.$nextTick(this._scrollDown())
  }
}
</script>

<style scoped>
.sc-message-list {
  height: 80%;
  overflow-y: auto;
  background-size: 100%;
  padding: 40px 0px;
}
</style>
