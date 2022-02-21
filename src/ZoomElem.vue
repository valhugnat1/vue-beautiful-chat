<template>
  <div>

    <div @mousedown.stop.prevent="zoomCarte" class="buttonMap" id="buttonZoom" style="display : none;"> 
      <img class="imgZoom" :src="'/api/assets/'+mapSetting.imgZoom" /> 
    </div>
    <div @mousedown.stop.prevent="dezoomCarte" class="buttonMap" id="buttonDezoom" style="display : none;">
      <img class="imgZoom" :src="'/api/assets/'+mapSetting.imgDezoom" > 
    </div>
    <div @mousedown.stop.prevent="quiteCarte" class="buttonMap" id="buttonQuite">
      <img class="imgZoom" :src="'/api/assets/'+mapSetting.imgCroix" >       
    </div>
    
    <p class="loadMsg" ref="txtLoad">Chargement de la carte ...</p>

    <div @touchstart="touchClic" ref="cadreZoom" id="zoomConteneur" :style="{'left':(widthWindow-mapSetting.sizeWidth)/2+'px', 'top':(heightWindow+115-mapSetting.sizeHeight)/2+'px', 'background': 'white'}"> <!--style="top:-430px; left:-1000px;"-->
      
       
      
      <PinchScrollZoom
        ref="zoomer"
        :width="mapSetting.sizeWidth"
        :height="mapSetting.sizeHeight"
        :scale="scale"
        @scaling="scalingHandler"
        @startDrag="scalingStartDrag"
        @stopDrag="scalingEndDrag"
        @dragging="draggingMove"
        style="border: 1px solid black"
        :within="false"
        :minScale="0.33"
        :translate-x="translateXBeg"
        :translate-y="translateYBeg">

        <img :src="'/api/assets/'+mapSetting.imgMap" @load="loadTxtHide" :width="mapSetting.sizeWidth" ref='imgPrin' :height="mapSetting.sizeHeight"/>
        <!-- <img :src="'https://studio.artybot.fr/api/assets/lovebot_marais/Plan_vierge_def.svg'" @load="loadTxtHide" :width="mapSetting.sizeWidth" ref='imgPrin' :height="mapSetting.sizeHeight"/>  -->

        <img id="positionUtilisateur" class="baliseUtili" ref="positionUtilisateur" :src="'/api/assets/'+mapSetting.imgBaliseUtili" 
        :style="{'top':-60-mapSetting.sizeHeight+'px', 'left':'0px', 'position': 'relative', 'witdh' : mapSetting.tailleUtili+'px'}"> 

        <img class="baliseMap" :class="suggestion.localisationLat + ' ' + suggestion.localisationLon" :id="'balise_'+suggestion.id + '  balise_' + idx" 
        v-for="(suggestion, idx) in suggestions" :key="idx" :ref="'baliseMap'+idx" :src="'/api/assets/' + mapSetting.imgBalisePlan + (idx+1) + mapSetting.imgBalisePlanExtention"
        :style="{'top':-60-mapSetting.sizeHeight+suggestion.localisationY-mapSetting.sizeBaliseHeight/2+'px', 
        'left':suggestion.localisationX-mapSetting.sizeBaliseWidth*idx-mapSetting.sizeBaliseWidth/2 - mapSetting.tailleUtili+'px', 'position': 'relative'}"> 
      </PinchScrollZoom>
    </div>
  </div>
</template>

<script>
import { Component, Vue } from "vue-property-decorator";
import PinchScrollZoom, { PinchScrollZoomEmitData } from "@coddicat/vue-pinch-scroll-zoom";
export default Vue.extend({
  data: () => ({
    scale: 0.7,
    widthWindow: window.innerWidth,
    heightWindow: window.innerHeight-115,
    placeTouch : "",
    posiPixelUserX: 0,
    posiPixelUserY:0,
    translateXBeg: 0,
    translateYBeg: 0
  }),
  components: {
    PinchScrollZoom,
  },
  props: 
  {
    colors: {
      type: Object,
      required: true
    },
    suggestions: {
      type: Array,
      default: () => []
    },
    mapSetting : {
      type: Object,
      required: true
    },
    mapOpen : {
      type: Boolean,
      required: true
    }
  },
  methods: {
    scalingHandler: function (e=false) { // Zoom
            var i = 0
            var elemHtmlBalise = this.$refs.cadreZoom.getElementsByClassName("baliseMap")
            
            for (let elem of elemHtmlBalise) {
              var witdhBalise = (this.mapSetting.sizeBaliseWidth / (this.$refs.zoomer.currentScale * 2))
              var heightBalise = (this.mapSetting.sizeBaliseHeight / (this.$refs.zoomer.currentScale * 2))

              elem.style.width = witdhBalise+'px'
              elem.style.left = (this.suggestions[i].localisationX-witdhBalise*i-this.mapSetting.sizeBaliseWidth - this.mapSetting.tailleUtili)+"px"
              elem.style.top = (this.suggestions[i].localisationY-60-this.mapSetting.sizeHeight-this.mapSetting.sizeBaliseHeight/2)+"px"
              i = i + 1
            }
    },
    draggingMove: function (e) { 
      //Drag
    },
    scalingEndDrag: function (e) { //Touch for balise 
      if (this.placeTouch != "")
      {
        this.$emit('sendSuggestion', this.placeTouch)
        this.placeTouch = ""
      }
    },
    reset: function () {
      this.$refs.zoomer.setData({
        scale: 1,
        originX: 0,
        originY: 0,
        translateX: 0,
        translateY: 0, 
        widthWindow: window.innerWidth,
        heightWindow: window.innerHeight-115

      });
    },
    touchClic: function (e) 
    {
      var p =  e.touches[0];
      var im = this.$refs.cadreZoom.getElementsByClassName("pinch-scroll-zoom__content")[0].getBoundingClientRect()
      var i = 0

      var xClient = (p.clientX-im.x)/this.$refs.zoomer.currentScale
      var yClient = (p.clientY-im.y)/this.$refs.zoomer.currentScale
      var sensibilitClicPosi = 50

      this.suggestions.forEach((item) =>{
        if (item.localisationX-sensibilitClicPosi < xClient && item.localisationX+sensibilitClicPosi > xClient && 
        item.localisationY - sensibilitClicPosi < yClient && item.localisationY + sensibilitClicPosi > yClient) 
        {
          this.placeTouch = item.choice
        }
        i = i + 1
      });
    }, 
    zoomCarte: function ()
    {
      this.$refs.zoomer.currentScale = this.$refs.zoomer.currentScale+0.25;
      this.scalingHandler()
    },
    dezoomCarte: function ()
    {
      if (this.$refs.zoomer.currentScale-0.25 > 0)
      {
        this.$refs.zoomer.currentScale = this.$refs.zoomer.currentScale-0.25;
        this.scalingHandler()
      }
    },
    quiteCarte: function ()
    {
      this.$emit('quiteMap')
    },
    checkPosi: function ()
    {
      
      try {
        if (this.mapSetting.actiPosition) {}
      } catch (error) {
        this.mapSetting.actiPosition = true
      }
      var baliseUtili = this.$refs.positionUtilisateur

      if (this.mapSetting.actiPosition && this.mapOpen) {
        navigator.geolocation.getCurrentPosition(this.successPosition, this.error)
        //navigator.geolocation.watchPosition(this.successPosition, this.error)
        
      } else {
        this.$refs.positionUtilisateur.style.left = "-1000px"
        this.$refs.positionUtilisateur.style.top = "-1000px"
      }

      if (this.mapOpen && baliseUtili !== undefined){
        setTimeout(() => {
          this.checkPosi()
        }, 3000)
      }
    },
    successPosition: function (pos) {
      var crdLat = pos.coords.latitude;
      var crdLon = pos.coords.longitude;
      var baliseUtili = this.$refs.positionUtilisateur

      //For test
      //var crdLat = 48.85723824457027
      //var crdLon = 2.35213476664655

      var resTempo = this.gps2pixel(crdLat, crdLon)
      this.posiPixelUserX = resTempo.x - this.mapSetting.tailleUtili/2
      this.posiPixelUserY = 100-60-this.mapSetting.sizeHeight+resTempo.y - this.mapSetting.tailleUtili/2

      baliseUtili.style.left =  this.posiPixelUserX + "px"
      baliseUtili.style.top = this.posiPixelUserY + "px"
    },
    error: function (err) {
      console.warn(`ERREUR (${err.code}): ${err.message}`);
      this.$refs.positionUtilisateur.style.left = "-1000px"
      this.$refs.positionUtilisateur.style.top = "-1000px"
    },
    gps2pixel: function (lat, long)
    {
      var coinHautGauche = { lat: this.mapSetting.positionLatLeftTop , long: this.mapSetting.positionLonLeftTop };
      var coinBasDroit = { lat: this.mapSetting.positionLatRightButtom, long: this.mapSetting.positionLonRightButtom };

        return {
            x: Math.round(this.mapSetting.sizeWidth * (long - coinHautGauche.long) / (coinBasDroit.long - coinHautGauche.long)),
            y: Math.round(this.mapSetting.sizeHeight * (lat - coinHautGauche.lat) / (coinBasDroit.lat - coinHautGauche.lat))
        };
    },
    loadTxtHide : function () {
      console.log("LOAD !!");
      this.$refs.txtLoad.style.cssText = 'display:none;';

      this.dezoomCarte()
    }
  },
  mounted() {
    this.scalingHandler()
    this.checkPosi()
    setTimeout(() => {
      var posiPixelX = (this.mapSetting.sizeWidth/2-this.posiPixelUserX)*this.scale
      var posiPixelY = (this.mapSetting.sizeHeight/2+this.posiPixelUserY)*this.scale
      if (this.posiPixelUserX != 0 && this.posiPixelUserY !=0 && posiPixelX>-10 && posiPixelX<this.mapSetting.sizeWidth && posiPixelY>-10 && posiPixelY<this.mapSetting.sizeHeight)
      {
        this.translateXBeg = (this.mapSetting.sizeWidth/2-this.posiPixelUserX)*this.scale
        this.translateYBeg = (this.mapSetting.sizeHeight/2+this.posiPixelUserY)*this.scale
      } else {
        this.$refs.positionUtilisateur.style.left = "-1000px"
        this.$refs.positionUtilisateur.style.top = "-1000px"
      }
    }, 2000);
  }
})
</script>
<style>
#zoomConteneur
{
  position: absolute;
  top: 60px;
}

.buttonMap {
  position: absolute;
  padding: 10px;
  /*background-color: #365ca7;*/
  border-radius: 10px;
  color: white;
  font-size: 18px;
  width: 40px;
  left: 5%;
  text-align: center;
  z-index: 1;
}

#buttonDezoom {
  top: 20%;
}

#buttonQuite {
  left: 80%;
  top: 10%;
}

#buttonZoom{
  top: 10%;
}

.imgZoom{
  width: 40px;
}

.loadMsg{
    position: absolute;
    top: 40%;
    left: 23%;
    z-index: 1;
    font-size: 20px;
    font-weight: bold;
    color: black;
}
</style>