<template>
    <div class="sc-suggestions-row" :style="{background: colors.messageList.bg}">
        <splide v-if="isSlider"
          :options="options"
          @splide:moved="moved"
        >
          <splide-slide v-for="(suggestion, idx) in suggestions" :key="suggestion.id" v-if="(currentHour >= suggestion.heureDebut && currentHour <= suggestion.heureFin) || suggestion.heureDebut === undefined || suggestion.heureFin === undefined" >
            <button v-bind:class="{ 'sc-suggestions-element-carrousel': suggestion.img !== undefined, 'sc-suggestions-element': suggestion.img === undefined }"  v-on:click="$emit('sendSuggestion', suggestion.choice)"
            :style="{borderColor: colors.sentMessage.text, color: colors.sentMessage.text}" :key="idx">

              <div class="sc-suggestions-element-img" 
              :style="[suggestion.description !== undefined ? 
              suggestion.telephone !== suggestion.localisation ? 
              suggestion.localisation !== undefined && suggestion.telephone !== undefined ? 
              {'color' : 'red', 'height' : '18vh', 'background-image': 'url(' + suggestion.img + ')'} : 
              {'color' : 'blue', 'height' : '24vh', 'background-image': 'url(' + suggestion.img + ')'} : 
              {'color' : 'green', 'height' : '30vh', 'background-image': 'url(' + suggestion.img + ')'} : 
              {'color' : 'white', 'height' : '40vh', 'background-image': 'url(' + suggestion.img + ')'}]" > </div>


              
              
              <div style="text-align: center; line-height: 40px; font-size: 16px;" :style="color : colors.sentMessage.text">{{suggestion.choice}}</div>

              <div style="margin-bottom: 0.5em; text-align: left; text-overflow: ellipsis; overflow: hidden; display: -webkit-box; -webkit-line-clamp: 4; -webkit-box-orient: vertical; 
              padding: 0px 8px 0px 8px; height: 5.8em; color: #585858;" v-if="suggestion.description !== undefined"> 
              <span v-if="suggestion.localisation !== undefined"> {{distanceGoogle[idx]}} {{tempsGoogle[idx]}} <br/></span> {{suggestion.description}}</div> 

              <div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" :style="color : colors.sentMessage.text" v-on:click.stop.prevent="getGoogleAdress(suggestion)" 
              v-if="suggestion.localisation !== undefined">Adresse 📍</div>

              <a :href="`tel:${suggestion.telephone}`" style="text-decoration: none;"><div style="padding : 0.5em; text-align: center; border: 1px solid #dedede; font-size: 16px;" :style="color : colors.sentMessage.text"
              v-on:click.stop.prevent="" v-if="suggestion.telephone !== undefined">☎️ {{suggestion.telephone}}</div></a>
            </button>
          </splide-slide>
        </splide>
        <button v-else class="sc-suggestions-element" v-for="(suggestion, idx) in suggestions" v-on:click="$emit('sendSuggestion', suggestion.choice)"
         :key="idx">{{suggestion.choice}}</button>
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
      for (var suggestion in self.suggestions) 
      {
        if (self.suggestions[i].latitude !== undefined)
        {
          self.distanceGoogle.push("📍 " + calcCrow(position.coords.latitude, position.coords.longitude, self.suggestions[i].latitude, self.suggestions[i].longitude) + " km")
        } else {
          self.distanceGoogle.push("")
        }
        i = i + 1
      }

    }

    function error(err) {
      console.warn(`ERREUR (${err.code}): ${err.message}`);
    }

    self = this
    console.log(this);



      let myPromise = new Promise(function(myResolve, myReject) {
        let recaptchaScript = document.createElement('script')
        recaptchaScript.setAttribute('src', 'http://maps.google.com/maps/api/js?key=AIzaSyAlyjo2L7BKSLJ4EQGC_qIiIspkUo391aI')
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
      );



    
  },
  methods: {
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
  border: 1px solid;
  border-radius: 15px;
  font-size: 14px;
  background: white;
  cursor: pointer;
  border-color: white !important;
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
