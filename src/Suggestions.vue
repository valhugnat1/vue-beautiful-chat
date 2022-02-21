<template>
    <div class="sc-suggestions-row" :style="{background: colors.messageList.bg}">
        <splide v-if="isSlider"
          :options="options"
          @splide:moved="moved"
        >
          <splide-slide v-for="(suggestion, idx) in suggestions" :key="suggestion.id" v-if="(currentHour >= suggestion.heureDebut && currentHour <= suggestion.heureFin) || suggestion.heureDebut === undefined || suggestion.heureFin === undefined" >
            
            <button v-bind:class="{ 'sc-suggestions-element-carrousel': suggestion.img !== undefined, 'sc-suggestions-element': suggestion.img === undefined }" 
             v-on:click="buttonClick(suggestion.cliqueGeneral, suggestion.boutonSavoirPlus, suggestion.choice, 'general')"
            :style="[colors.carrousel !== undefined ? {'borderColor': colors.sentMessage.text, 'color': colors.sentMessage.text, 'background': colors.carrousel.bg} : 
            {'borderColor': colors.sentMessage.text, 'color': colors.sentMessage.text, 'background': 'white'} ]" :key="idx">

              <div class="sc-suggestions-element-img" :style="{'height' : setHeightImg(suggestion), 'background-image': 'url(' + suggestion.img + ')'}" > </div>

              <div style="text-align: center; line-height: 40px; font-size: 16px;" :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.title} : {'color': '#295ca3'}]">{{suggestion.choice}}</div>

              <div style="margin-bottom: 0.5em; text-align: left; text-overflow: ellipsis; overflow: hidden; display: -webkit-box; -webkit-line-clamp: 4; -webkit-box-orient: vertical; 
              padding: 0px 8px 0px 8px; height: 5.8em; " :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.textBody} : {'color': 'black'}]" v-if="suggestion.description !== undefined"> 
              <span v-if="suggestion.localisation !== undefined"> {{distanceGoogle[idx]}} {{tempsGoogle[idx]}} <br/></span> {{suggestion.description}}</div> 

              <div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" 
              :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.title} : {'color': '#295ca3'}]" v-on:click.stop.prevent="getGoogleAdress(suggestion)" 
              v-if="suggestion.localisation !== undefined">Adresse 📍</div>

              <a :href="`tel:${suggestion.telephone}`" style="text-decoration: none;"><div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" 
              :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.title} : {'color': '#295ca3'}]"
               v-if="suggestion.telephone !== undefined">☎️ {{suggestion.telephone}}</div></a>

              <div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.title} : {'color': '#295ca3'}]" 
              v-on:click.stop.prevent="buttonClick(suggestion.cliqueGeneral, suggestion.boutonSavoirPlus, suggestion.choice, 'special')" 
              v-if="suggestion.boutonSavoirPlus">A savoir plus ➕</div>

              <div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" 
              :style="[ colors.carrousel !== undefined ? {color : colors.carrousel.title} : {'color': '#295ca3'}]" v-on:click.stop.prevent="getlinkExtern(suggestion.siteWeb)" 
              v-if="suggestion.siteWeb !== undefined">Site Web</div> 

            </button>
          </splide-slide>
        </splide>
        <button v-else class="sc-suggestions-element" v-for="(suggestion, idx) in suggestions" v-on:click="$emit('sendSuggestion', suggestion.choice)"
         :key="idx" :style="{'color': colors.userInput.text, 'background': colors.userInput.bg}">{{suggestion.choice}}</button>
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
          fixedWidth: "420px",
          autoHeight: true
        },
        distanceGoogle: [],
        tempsGoogle: [],
        latUser: 0,
        LonUser:0,
        heightImage: [],
        currentHour: new Date().getHours()
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
    }
  },
  mounted() {

    console.log("colors", this.colors)

    function getGoogleDistTemps(position) {

      var i = 0
      var originsList = []
      var destinationsList = []
      for (var suggestion in self.suggestions) 
      {
        originsList.push({lat: position.coords.latitude, lng: position.coords.longitude})
        destinationsList.push({lat: self.suggestions[i].latitude, lng: self.suggestions[i].longitude})
        i = i + 1
      }

      //Find the distance
      var distanceService = new google.maps.DistanceMatrixService();
      distanceService.getDistanceMatrix({
        origins: originsList,
        destinations: destinationsList,
        travelMode: google.maps.TravelMode.WALKING,
        unitSystem: google.maps.UnitSystem.METRIC,
        durationInTraffic: true,
        avoidHighways: false,
        avoidTolls: false
      },
      function (response, status) {
        if (status !== google.maps.DistanceMatrixStatus.OK) {
          console.log('Error:', status);
        } else {
          
          response.rows.forEach(function (elem, i)
          {
            self.distanceGoogle.push("📍 " + elem.elements[i].distance.text)
            self.tempsGoogle.push("🚶 " + elem.elements[i].duration.text)
          });
          console.log("distance & temps google : ", self.distanceGoogle, self.tempsGoogle, " // infoG : ", response)
        }
      });
        
    }

    function getDistance(position) {
      
      function calcCrow (lat1, lon1, lat2, lon2) {
        var R = 6371; // km
        var dLat = toRad(lat2-lat1);
        var dLon = toRad(lon2-lon1);
        var lat1 = toRad(lat1);
        var lat2 = toRad(lat2);

        var a = Math.sin(dLat/2) * Math.sin(dLat/2) +
          Math.sin(dLon/2) * Math.sin(dLon/2) * Math.cos(lat1) * Math.cos(lat2); 
        var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a)); 
        var d = R * c;
        return d.toFixed(2);
      }
      function toRad (Value) {
        return Value * Math.PI / 180;
      }
      
      var i = 0
      for (var suggestion in selfSuggestions) 
      {
        if (selfSuggestions[i].latitude !== undefined)
        {
          selfDistanceGoogle.push("📍 " + calcCrow(position.coords.latitude, position.coords.longitude, selfSuggestions[i].latitude, selfSuggestions[i].longitude) + " km")
        } else {
          selfDistanceGoogle.push("")
        }
        i = i + 1
      }

    }

    function error(err) {
      console.warn(`ERREUR (${err.code}): ${err.message}`);
    }

    var selfSuggestions = []
    var selfDistanceGoogle = []
    setTimeout(() => {
      selfSuggestions = this.suggestions
      selfDistanceGoogle = this.distanceGoogle


      navigator.geolocation.getCurrentPosition(getDistance, error)
    }, 2000)

    /*let myPromise = new Promise(function(myResolve, myReject) {
      let recaptchaScript = document.createElement('script')
      recaptchaScript.setAttribute('src', 'http://maps.google.com/maps/api/js?key={putKeyHere}')
      document.head.appendChild(recaptchaScript)
      myResolve();
    });
    
    myPromise.then(
      setTimeout(() => {
          if (this.suggestions[0].apigoogle == "oui") {
            navigator.geolocation.getCurrentPosition(getGoogleDistTemps)
          } else {
            navigator.geolocation.getCurrentPosition(getDistance)
          }
        }, 2000)
    );*/



    
  },
  methods: {
    buttonClick : function (generalClickActive, speClickActive, msg, path) {

      if (generalClickActive == undefined) generalClickActive = true
      if (speClickActive == undefined) speClickActive = false 

      console.log(generalClickActive, speClickActive, msg, path);
      if ((path == "general" && generalClickActive) || (path == "special" && speClickActive ) )
      {
        this.$emit('sendSuggestion', msg)
      }
      
    },
    setHeightImg : function (suggestion) {

      var elemNoneDisplay = [suggestion.description, suggestion.boutonSavoirPlus, suggestion.telephone, suggestion.localisation].filter(x => x==undefined).length
      if (elemNoneDisplay == 4) elemNoneDisplay = 5 
      if (window.innerWidth > 450) 
      {
        return (90 + elemNoneDisplay*30).toString() + "px"
      } else {
        return (15 + elemNoneDisplay*3.5).toString() + "vh"
      }
    },
    getGoogleAdress : function (suggestion) {

      function success(pos) {
        var crd = pos.coords;
      }

      function error(err) {
        console.warn(`ERREUR (${err.code}): ${err.message}`);
      }

      var url = "https://www.google.com/maps/search/?api=1&query="+(suggestion.choice+ ", " + suggestion.localisation).replaceAll(" ","+").replaceAll(",", "%2C")
      console.log("Test adress !", suggestion.localisation, url);
      window.open(url, '_blank');

      navigator.geolocation.getCurrentPosition(success, error)
    },
    getlinkExtern : function (suggestion) {

      window.open(suggestion, '_blank');
    }
  }, create() {
    if (typeof this.colors.carrousel !== 'undefined') {
      this.colors.carrousel.bg = "white"
      this.colors.carrousel.title = "#295ca3"
      this.colors.carrousel.textBody = "black"

      console.log(this.colors.carrousel)
    } else {
      console.log("Set default valor")
    }
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
  border-radius: 15px;
  font-size: 14px;
  background: white;
  cursor: pointer;    
  box-shadow: -2px 3px rgb(247 198 197 / 80%);
  border: #295ca3 2px solid;
  color : #295ca3;

}

.sc-suggestions-element-carrousel {
  font-size: 14px;
  background: white;
  cursor: pointer;
  border-color: white !important;
  padding: 0px 0px 0px 0px; 
  width: 70vw; 
  margin: 5px 100% 30px calc(10px + 12vw); 
  color: black; 
  font-family: Roboto,Arial,sans-serif; 
  border-radius: 15px 15px 0px 0px; 
}

.sc-suggestions-element-img
{
  border-radius: 15px 15px 0px 0px;
  margin: 10px;
  background-position: center;
  background-size: contain;
}

@media (min-width: 450px) {

  .sc-suggestions-element-carrousel {
      background: white;
      cursor: pointer;
      border-color: white !important;

      padding: 0px 0px 0px 0px; 
      width: 270px; 
      margin: 5px 100% 30px 50px; 
      color: black; 
      font-family: Roboto,Arial,sans-serif; 
      border-radius: 15px 15px 0px 0px; 
  }
}
</style>
