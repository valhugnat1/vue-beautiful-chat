<template>
    <div class="sc-suggestions-row" :style="{background: colors.messageList.bg}">
        <splide v-if="isSlider"
          :options="options"
          @splide:moved="moved"
        >
          <splide-slide v-for="(suggestion, idx in suggestions" :key="suggestion.id">
            <button class="sc-suggestions-element"  v-on:click="$emit('sendSuggestion', suggestion.choice)"
            :style="{borderColor: colors.sentMessage.text, color: colors.sentMessage.text}" :key="idx">
              <img :src="suggestion.img" :alt="suggestion.choice">
              <div style="margin-bottom: 1rem; text-align: left;">{{suggestion.choice}}</div>
            </button>
          </splide-slide>
        </splide>
        <button v-else class="sc-suggestions-element" v-for="(suggestion, idx) in suggestions" v-on:click="$emit('sendSuggestion', suggestion.choice)"
        :style="{borderColor: colors.sentMessage.text, color: colors.sentMessage.text}" :key="idx">{{suggestion.choice}}</button>
    </div>
</template>

<script>
import { Splide, SplideSlide } from '@splidejs/vue-splide'
export default {
  	components: {
  		Splide,
        SplideSlide,
    },
    data() {
      return {
          options: {
              rewind: true,
              gap   : '1rem',
              fixedWidth: "370px",
              autoHeight: true
          },
      };
    },
    props: {
        suggestions: {
            type: Array,
            default: () => []
        },
        colors: {
            type: Object,
            required: true
        },
    },
  computed: {
    isSlider() {
     let result  = false;
     try {
        result =  this.suggestions && this.suggestions.lenght != 0 && this.suggestions[0].img;
     } catch {
         result = false
     }
     return result
    },
   }
}
</script>

<style>
.sc-suggestions-row {
  text-align: center;
  background: inherit;
}

.sc-suggestions-element {
  margin: 3px;
  padding: 5px 10px 5px 10px;
  border: 1px solid;
  border-radius: 15px;
  font-size: 14px;
  background: white;
  cursor: pointer;
  border-color: white !important;
}
</style>
