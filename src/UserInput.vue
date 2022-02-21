<template>
  <div>
    <ZoomElem v-if="mapShow && !onlyButton" v-on:quiteMap="_quiteMap" v-on:sendSuggestion="_submitSuggestion" :mapSetting="mapSetting" :colors="colors" :suggestions="suggestions" :mapOpen="mapShow" />
    <Suggestions v-if="suggestions && onlyButton" :suggestions="suggestions" v-on:sendSuggestion="_submitSuggestion" :colors="colors" />
    <div v-if="file && onlyButton" class='file-container' :style="{backgroundColor: colors.userInput.text, color: colors.userInput.bg}" >
      <span class='icon-file-message'><img :src="icons.file.img"  :alt="icons.file.name" height="15" /></span>
      {{file.name}}
      <span class='delete-file-message' @click="cancelFile()"><img :src="icons.closeSvg.img" :alt="icons.closeSvg.name" height="10" title='Remove the file' /></span>
    </div>
    <form class="sc-user-input" :class="{active: inputActive}" :style="{background: colors.userInput.bg}" v-if="onlyButton == false && showInputText">
      <div
        role="button"
        tabIndex="0"
        @focus="setInputActive(true)"
        @blur="setInputActive(false)"
        @keydown="handleKey"
        contentEditable="true"
        :placeholder="placeholder"
        class="sc-user-input--text"
        ref="userInput"
        :style="{color: colors.userInput.text}"
        @focusUserInput="focusUserInput()"
      >
      </div>
      <div class="sc-user-input--buttons">
        <div class="sc-user-input--button"></div>
        <button v-if="showPhotos" v-on:click.prevent="_showIndicator('!photo')" class="sc-user-input--button-icon-wrapper" id="showPhoto" tooltip="Photo">
          <img class="img-indicator-photo" src="artybot/photo.svg" />
        </button>
        <button v-if="showMap" v-on:click.prevent="_showIndicator('!map')" class="sc-user-input--button-icon-wrapper" id="showMap" tooltip="Map">
          <img class="img-indicator-map" src="artybot/local.svg" />
        </button>
          <button v-if="showCredits" v-on:click.prevent="_showIndicator('!credits')" class="sc-user-input--button-icon-wrapper" id="showCredits" tooltip="Credits">
          <img class="img-indicator-credits" src="artybot/lettre_coeur.svg" />
        </button>
        <button v-if="showHelp" v-on:click.prevent="_showIndicator('!help')" class="sc-user-input--button-icon-wrapper" id="showHelp" tooltip="Help">
          <img class="img-indicator-help" src="artybot/aide.svg" />
        </button>
        <div v-if="showEmoji && !isEditing" class="sc-user-input--button">
          <EmojiIcon :onEmojiPicked="_handleEmojiPicked" :color="colors.userInput.text" />
        </div>
        <div v-if="showFile && !isEditing" class="sc-user-input--button">
          <FileIcons :onChange="_handleFileSubmit" :color="colors.userInput.text" />
        </div>
        <div v-if="isEditing" class="sc-user-input--button">
          <user-input-button @click.native.prevent="_editFinish" :color="colors.userInput.text" tooltip="cancel">
            <icon-cross />
          </user-input-button>
        </div>
        <div class="sc-user-input--button" v-if="showSend">
          <user-input-button @click.native.prevent="_editText" v-if="isEditing" :color="colors.userInput.text" tooltip="Edit">
            <icon-ok />
          </user-input-button>
          <user-input-button @click.native.prevent="_submitText" v-else :color="colors.userInput.text" tooltip="Send">
            <icon-send />
          </user-input-button>
        </div>
      </div>
    </form>


    <div id="bottomButton" class="sc-user-input" :class="{active: inputActive}" :style="{background: colors.userInput.bg}" 
    v-if="onlyButton == false && (this.imgUserInput.legale != '' || this.imgUserInput.contenuSpecial1 != '' || this.imgUserInput.contenuSpecial2 != '' || this.imgUserInput.contenuSpecial3 != '' || this.imgUserInput.menu != '' || this.imgUserInput.carte != '' )">
      <div id="diplayButton">
        <div v-if="twoOrLessBtn()" id="putButtonOnRight"></div>
        <!-- LEGAL -->
        <div v-if="imgUserInput.legale != '' && imgUserInput.legale != undefined">
          <div :class="[showTypingIndicator ? 'cacheButtom' : ''] " >
            <button @click.prevent="clickButton('Politique de confidentialité', !(showTypingIndicator), false)" class="sc-user-input--button-icon-wrapper" id="extraCarrousel" >
              <img class="img-indicator-map" :src="imgUserInput.legale"/>
            </button>
          </div>
        </div>
        <!-- MENU -->
        <div v-if="imgUserInput.menu != '' && imgUserInput.menu != undefined">
          <div :class="[showTypingIndicator || menuNotPass() ? 'cacheButtom' : ''] ">
            <button class="sc-user-input--button-icon-wrapper" id="extraCarrousel" @click="clickButton('Menu', !(showTypingIndicator || menuNotPass()), true)" >
              <img class="img-indicator-map" :src="imgUserInput.menu" />
            </button>
          </div>
        </div>
        <!-- contenuSpecial1 --> <!-- || beginNotPass() -->
        <div v-if="imgUserInput.contenuSpecial1 != '' && imgUserInput.contenuSpecial1 != undefined">
          <div :class="[showTypingIndicator || menuNotPass()  ? 'cacheButtom' : '']">
            <button @click="clickButton('Bonnes adresses', !(showTypingIndicator || menuNotPass() ), false)" class="sc-user-input--button-icon-wrapper" id="extraCarrousel" >
              <img class="img-indicator-map" :src="imgUserInput.contenuSpecial1" />
            </button>
          </div>
        </div>
        <!-- contenuSpecial2 -->
        <div v-if="imgUserInput.contenuSpecial2 != '' && imgUserInput.contenuSpecial2 != undefined">
          <div :class="[showTypingIndicator || menuNotPass() ? 'cacheButtom' : '']"> 
            <button @click="clickButton('En savoir plus', !(showTypingIndicator || menuNotPass() ), false)" class="sc-user-input--button-icon-wrapper" id="extraCarrousel" >
              <img class="img-indicator-map" :src="imgUserInput.contenuSpecial2" />
            </button>
          </div>
        </div>
        <!-- contenuSpecial3 -->
        <div v-if="imgUserInput.contenuSpecial3 != '' && imgUserInput.contenuSpecial3 != undefined">
          <div :class="[showTypingIndicator || menuNotPass() ? 'cacheButtom' : '']">
            <button @click="clickButton('En savoir +', !(showTypingIndicator || menuNotPass()), false)" class="sc-user-input--button-icon-wrapper" id="extraCarrousel" >
              <img class="img-indicator-map" :src="imgUserInput.contenuSpecial3" />
            </button>
          </div>
        </div>
        <!-- MAP -->
        <div v-if="imgUserInput.carte != '' && imgUserInput.carte != undefined">
          <div :class="[!(mapSetting.exist && calculNbRepBeforeFct() && !showTypingIndicator) ? 'cacheButtom' : '']">
            <button @click="clickButton('Plan', (mapSetting.exist && calculNbRepBeforeFct() && !showTypingIndicator), false)" class="sc-user-input--button-icon-wrapper" id="showMap" tooltip="Map">
              <img class="img-indicator-map" :src="imgUserInput.carte" />
            </button>
          </div>
        </div>
        <!-- <div v-if="twoOrLessBtn()" id="putButtonOnRight" style="width: 15%"></div> -->
      </div>
    </div>
  </div>
</template>


<script>
import EmojiIcon from './icons/EmojiIcon.vue'
import FileIcons from './icons/FileIcons.vue'
import UserInputButton from './UserInputButton.vue'
import Suggestions from './Suggestions.vue'
import FileIcon from './assets/file.svg'
import CloseIconSvg from './assets/close.svg'
import store from "./store/"
import IconCross from "./components/icons/IconCross.vue";
import IconOk from "./components/icons/IconOk.vue";
import IconSend from "./components/icons/IconSend.vue";
import { length } from 'file-loader'
import ZoomElem from "./ZoomElem.vue";

export default {
  components: {
    EmojiIcon,
    FileIcons,
    UserInputButton,
    Suggestions,
    IconCross,
    IconOk,
    IconSend,
    ZoomElem
  },
  props: {
    icons:{
      type: Object,
      required: false,
      default: function () {
        return {
          file:{
            img: FileIcon,
            name: 'default',
          },
          closeSvg:{
            img: CloseIconSvg,
            name: 'default',
          },
        }
      }
    },
    showEmoji: {
      type: Boolean,
      default: () => false
    },
    showPhotos: {
      type: Boolean,
      default: () => false
    },
    showMap: {
      type: Boolean,
      default: () => false
    },
    showCredits: {
      type: Boolean,
      default: () => false
    },
    showHelp: {
      type: Boolean,
      default: () => false
    },
    showSend: {
      type: Boolean,
      default: false
    },
    suggestions: {
      type: Array,
      default: () => []
    },
    // suggestionSlider: {
    //   type: Boolean,
    //   default: () => false
    // },
    showFile: {
      type: Boolean,
      default: () => false
    },
    onSubmit: {
      type: Function,
      required: true
    },
    placeholder: {
      type: String,
      default: 'Write something...'
    },
    colors: {
      type: Object,
      required: true
    },
    mapSetting : {
      type: Object,
      required: true
    },
    imgUserInput : {
      type: Object,
      required: false
    },
    device : {
      type: Object,
      required: false
    },
    showInputText : {
      type: Boolean,
      required: true
    },
    onlyButton: {
      type: Boolean,
      required: true
    },
    messages: {
      type: Array,
      required: true
    },
    showTypingIndicator: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      file: null,
      inputActive: false,
      store,
      mapShow: false,
      loadMsgMapGood: false, 
      waitongMapCount: -10, 
      msgSpe: ""
    }
  },
  updated() {
    if (!this.onlyButton && this.messages != undefined && this.messages.length > 4)
    {
      if (this.waitongMapCount+2 == this.messages.length) {
          this._submitSuggestion("ok") //Valide le message indiquand que la carte arrive 
      }

      if (this.waitongMapCount+4 == this.messages.length && this.msgSpe == "map") {
          this.waitongMapCount = -10;
          this.mapShow = !this.mapShow // Affiche la carte
          this.msgSpe = ""
      }
    }
  },
  methods: {
    twoOrLessBtn() {
      var r = [this.imgUserInput.legale != '' && this.imgUserInput.legale != undefined, this.imgUserInput.menu != '' && this.imgUserInput.menu != undefined,
      this.imgUserInput.contenuSpecial1 != '' && this.imgUserInput.contenuSpecial1 != undefined, this.imgUserInput.contenuSpecial2 != '' && this.imgUserInput.contenuSpecial2 != undefined,
      this.imgUserInput.contenuSpecial3 != '' && this.imgUserInput.contenuSpecial3 != undefined, this.imgUserInput.carte != '' && this.imgUserInput.carte != undefined]

      const countOccurrences = (arr, val) => arr.reduce((a, v) => (v === val ? a + 1 : a), 0);
      if (countOccurrences(r, true) <= 2) return true
      else return false

    },
    clickButton (msg, cond, okMsgAfter) {
      if (cond && msg =="Plan"){
        this.modifStatutMap()
      } 
      else if (cond) {
        this._showIndicator(msg)

      if (okMsgAfter) {
        this.waitongMapCount = this.messages.length;
      }
      }
    },
    menuNotPass () { // pour l'affichage de des bouttons dans la ligne du bas 
      var menuPassLoop = false
      var beginPass = false 
      var end = false 
      try {
        this.device.surveys[0].questions.forEach(elem =>{
          
          if (elem.id == this.device.surveys[0].chat.beginDefaultMsgId) beginPass = true
          if (elem.id == this.device.surveys[0].chat.menuId) menuPassLoop = true
          if (beginPass == menuPassLoop && elem.id == this.device.question.id ) {end = true; return false;}
        });
      } catch (e) {
        return true
      }
      if (!end) return true
    },
    beginNotPass () { // pour l'affichage de des bouttons dans la ligne du bas 
      var beginPass = false 
      var end = false 
      try {
        this.device.surveys[0].questions.forEach(elem =>{
          if (elem.id == this.device.surveys[0].chat.beginDefaultMsgId) beginPass = true
          if (beginPass  && elem.id == this.device.question.id ) {end = true; return false;}
        });
      } catch (e) {
        return true
      }
      if (!end)  return true
    },
    calculNbRepBeforeFct () {
      var count = 0;
      if (!this.onlyButton && this.messages.length > 1)
      {
        this.messages.forEach(element => {
          if (element.author == "me") count = count + 1
        });
      }
      //return this.mapSetting.nbMsgAvantAccesMap <= count 
      try {
        return this.suggestions[0].baliseAcces
      } catch (e) {
        return false
      }
    },
    modifStatutMap () {
      if (this.mapShow)
      {
        this.mapShow = !this.mapShow // Cache la carte
      } else {
        this._showIndicator('Plan') // Envoi le message pour demander d'afficher la carte
        this.waitongMapCount = this.messages.length;
        this.msgSpe = 'map'
      }
    },
    cancelFile () {
      this.file = null
    },
    setInputActive (onoff) {
      this.inputActive = onoff
    },
    handleKey (event) {
      if (event.keyCode === 13 && !event.shiftKey) {
        if (!this.isEditing){
          this._submitText(event);
        }
        else{
          this._editText(event);
        }
        this._editFinish();
        event.preventDefault()
      }
      else if (event.keyCode === 27) {
        this._editFinish();
        event.preventDefault();
      }

      this.$emit('onType')
    },
    focusUserInput() {
      this.$nextTick(() => {
        this.$refs.userInput.focus();
      })
    },
    _submitSuggestion(suggestion) {
      if (this.mapShow)
      {
        this.mapShow = !this.mapShow // Cache la carte

      }
      this.onSubmit({author: 'me', type: 'text', data: { text: suggestion }})
    },
    _quiteMap() {
      this.mapShow = false // Cache la carte
    },
    _checkSubmitSuccess (success) {
      if (Promise !== undefined) {
        Promise.resolve(success).then(function (wasSuccessful) {
          if (wasSuccessful === undefined || wasSuccessful) {
            this.file = null
            this.$refs.userInput.innerHTML = '';
          }
        }.bind(this));

      } else {
        this.file = null
        this.$refs.userInput.innerHTML = '';
      }
    },
    _submitText (event) {
      const text = this.$refs.userInput.textContent
      const file = this.file
      if (file) {
        this._submitTextWhenFile(event, text, file)
      } else {
        if (text && text.length > 0) {
          this._checkSubmitSuccess(this.onSubmit({
            author: 'me',
            type: 'text',
            data: { text }
          }));
        }
      }
    },
    _submitTextWhenFile(event, text, file) {
      if (text && text.length > 0) {
        this._checkSubmitSuccess(this.onSubmit({
          author: 'me',
          type: 'file',
          data: { text, file }
        }))
      } else {
        this._checkSubmitSuccess(this.onSubmit({
          author: 'me',
          type: 'file',
          data: { file }
        }))
      }
    },
    _editText (event) {
      const text = this.$refs.userInput.textContent;
      if (text && text.length) {
        this.$emit('edit', {
          author: 'me',
          type: 'text',
          id: store.editMessage.id,
          data: { text }
        });
        this._editFinish();
      }
    },
    _handleEmojiPicked (emoji) {
      this._checkSubmitSuccess(this.onSubmit({
        author: 'me',
        type: 'emoji',
        data: { emoji }
      }))
    },
    _handleFileSubmit (file) {
      this.file = file
    },
    _editFinish(){
      this.store.editMessage = null;
    },
    _showIndicator(indicatorName) {
      this.onSubmit({author: 'me', type: 'raw', data: { text: indicatorName }})
    }
  },
  watch: {
    editMessageId(m){
      if (store.editMessage != null && store.editMessage != undefined){
        this.$refs.userInput.focus();
        this.$refs.userInput.textContent = store.editMessage.data.text;
      } else {
        this.$refs.userInput.textContent = '';
      }
    }
  },
  computed: {
    editMessageId() {
      return this.isEditing && store.editMessage.id;
    },
    isEditing() {
      return store.editMessage && store.editMessage.id
    },
    calculNbRepBefore() {
      var count = 0;
      if (!this.onlyButton && this.messages.length > 4)
      {
        this.messages.forEach(element => {
          if (element.author == "me") count = count + 1
        });
      }
      return this.mapSetting.nbMsgAvantAccesMap <= count 
    }, 
    mountedMapCheckMsg() {
      if (!this.onlyButton && this.messages != undefined && this.messages.length > 4)
      {
        if (this.messages.at(-3).data.text == "Voici la carte") {
          this.$forceUpdate();
          return true
        } else {
          return false
        }
      }
    },
  },
  mounted() {
    console.log("2d2bcc3a060bfe7b3a0db903f3520ee286e8f56184c49af742836d67e32ce3fc");
    
    this.$root.$on('focusUserInput', () => {
      if (this.$refs.userInput) {
        this.focusUserInput()
      }
    })
  }
}
</script>

<style>
.sc-user-input {
  min-height: 55px;
  margin: 0px;
  position: relative;
  bottom: 0;
  display: flex;
  background-color: #f4f7f9;
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
  transition: background-color 0.2s ease, box-shadow 0.2s ease;
}

.sc-user-input--text {
  width: 300px;
  resize: none;
  border: none;
  outline: none;
  border-bottom-left-radius: 10px;
  box-sizing: border-box;
  padding: 18px;
  font-size: 15px;
  font-weight: 400;
  line-height: 1.33;
  white-space: pre-wrap;
  word-wrap: break-word;
  color: #565867;
  -webkit-font-smoothing: antialiased;
  max-height: 200px;
  overflow: scroll;
  bottom: 0;
  overflow-x: hidden;
  overflow-y: auto;
}

.sc-user-input--text:empty:before {
  content: attr(placeholder);
  display: block; /* For Firefox */
  /* color: rgba(86, 88, 103, 0.3); */
  filter: contrast(15%);
  outline: none;
  cursor: text;
}

.sc-user-input--buttons {
  width: 100px;
  position: absolute;
  right: 30px;
  height: 100%;
  display: flex;
  justify-content: flex-end;
}

.sc-user-input--button:first-of-type {
  width: 40px;
}

.sc-user-input--button {
  width: 30px;
  height: 55px;
  margin-left: 2px;
  margin-right: 2px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.sc-user-input.active {
  box-shadow: none;
  background-color: white;
  box-shadow: 0px -5px 20px 0px rgba(150, 165, 190, 0.2);
}

.sc-user-input--button label {
  position: relative;
  height: 24px;
  padding-left: 3px;
  cursor: pointer;
}

.sc-user-input--button label:hover path {
  fill: rgba(86, 88, 103, 1);
}

.sc-user-input--button input {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  z-index: 99999;
  height: 100%;
  opacity: 0;
  cursor: pointer;
  overflow: hidden;
}

.file-container {
  background-color: #f4f7f9;
  border-top-left-radius: 10px;
  padding: 5px 20px;
  color: #565867;
}

.delete-file-message {
  font-style: normal;
  float: right;
  cursor: pointer;
  color: #c8cad0;
}

.delete-file-message:hover {
  color: #5d5e6d;
}

.icon-file-message {
  margin-right: 5px;
}

.img-indicator-photo {
  width: 32px;
  height: 32px;
}

.img-indicator-map {
  width: 32px;
  height: 32px;
}

.img-indicator-credits {
  width: 32px;
  height: 32px;
}

.img-indicator-help {
  width: 32px;
  height: 32px;
}

.credit {
  font-size: 7px !important;
}

#bottomButton
{
  display: flex;
  flex-wrap: nowrap;
  justify-content: center;
}

#diplayButton
{
  position: absolute;
  height: 100%;
  width: 100%;
  right: 0px;
  justify-content: space-evenly;
  display: flex;
  border-top: 1px solid #00000036;
  padding-top: 3px;
}

.cacheButtom
{
  background-color: white;
  opacity: 0.3;
}

#diplayButton img 
{
  height: 45px;
  width: 55px;
  object-fit: scale-down;
}

#putButtonOnRight
{
  height: 100%;
  width: 70%;
}
</style>
