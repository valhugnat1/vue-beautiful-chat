<template>
  <div>
    <zoomElem/>

    <!--<div id="backgroundFlou"></div>
    <div id="cadreForMapMenu" ref="cadreForMapMenu" @mousedown="initialClick" @mouseup="endClick" @mousemove="move" @touchstart="initialClickTel" @touchmove.passive="moveTel">
      <img id="imgMap" ref="imgMap" :src="'/api/assets/'+mapSetting.imgMap" > 
      <img id="positionUtilisateur" class="baliseTest" ref='positionUtilisateur' :src="'/api/assets/'+mapSetting.imgBaliseUtili"> 
      <img class="baliseTest " :class="suggestion.localisationLat + ' ' + suggestion.localisationLon" :id="'baliseTest'+suggestion.id" v-for="(suggestion, idx) in suggestions" :key="idx" :ref="'baliseTest'+idx" :src="'/api/assets/'+mapSetting.imgBalisePlan"> 
     
    </div>
    <div @mousedown="zoomCarte" class="doubleButtonZoom" id="buttonZoom"> 
      <img class="imgZoom" :src="'/api/assets/'+mapSetting.imgZoom" > 
    </div>
    <div @mousedown="dezoomCarte" id="buttonDezoom" class="doubleButtonZoom">
      <img class="imgZoom" :src="'/api/assets/'+mapSetting.imgDezoom" > 
      
    </div>-->
  </div>
</template>

<script>

function gps2pixel(lat, long, positionLatLeftTop, positionLatRightButtom, positionLonLeftTop, positionLonRightButtom, width, height)
{
  var coinHautGauche = { lat: positionLatLeftTop , long: positionLonLeftTop };
  var coinBasDroit = { lat: positionLatRightButtom, long: positionLonRightButtom };
  

    return {
        x: Math.round(width * (long - coinHautGauche.long) / (coinBasDroit.long - coinHautGauche.long)),
        y: Math.round(height * (lat - coinHautGauche.lat) / (coinBasDroit.lat - coinHautGauche.lat))
    };
}


function verifClicButton (ListNomLieu, listLoca, xClient, yClient, sensibilitClicPosi, positionLatLeftTop, positionLatRightButtom, positionLonLeftTop, positionLonRightButtom, width, height) {

  var locPix = []
  var i = 0
  var resLieu = "None"

  listLoca.forEach(function(item){

    var resTempo = gps2pixel(item[0], item[1], positionLatLeftTop, positionLatRightButtom, positionLonLeftTop, positionLonRightButtom, width, height)
    locPix.push([resTempo.x, resTempo.y])

  });
  


  locPix.forEach(function(item){
    if (item[0]-sensibilitClicPosi < xClient && item[0]+sensibilitClicPosi > xClient && item[1] - sensibilitClicPosi < yClient && item[1] + sensibilitClicPosi > yClient) 
    {
      resLieu =  ListNomLieu[i]
    }
    i = i + 1
  });
  return resLieu
}

function valueZoomTranslat (img, cdr, imgMarg, indiZoom, ratio) {

  var ret = ((img  + indiZoom*ratio) * (imgMarg-cdr/2) / img) - (imgMarg-cdr/2)
  return ret
}

function baliseSurLaCarte (baliseImg, baliseUtili, newX, newY, listLoca, mapSetting, ImgNaturalWidth, ImgNaturalHeight, ImgWidth = 1, ImgHeight = 1, xMov = true, YMov = true) {

  console.log(baliseImg, listLoca);
  function success(pos) {
    var crdLat = pos.coords.latitude;
    var crdLon = pos.coords.longitude;
    console.log(pos)

    //var crdLat = 48.85723824457027
    //var crdLon = 2.35213476664655

    var resTempo = gps2pixel(crdLat, crdLon, mapSetting.positionLatLeftTop, mapSetting.positionLatRightButtom, mapSetting.positionLonLeftTop, mapSetting.positionLonRightButtom,  ImgNaturalWidth, ImgNaturalHeight)
    

    var xBalise = (newX + resTempo.x* (ImgWidth/ImgNaturalWidth) - baliseUtili.width / 2) + "px"
    var yBalise = (newY  + resTempo.y* (ImgHeight/ImgNaturalHeight)- baliseUtili.height / 2) + "px"

    if (xMov) baliseUtili.style.left =  xBalise
    if (YMov) baliseUtili.style.top = yBalise 
  }

  function error(err) {
    console.warn(`ERREUR (${err.code}): ${err.message}`);
  }


  var i = 0
  listLoca.forEach(function(item){
    console.log(item[0],item[1]);
    var resTempo = gps2pixel(item[0],item[1], mapSetting.positionLatLeftTop, mapSetting.positionLatRightButtom, mapSetting.positionLonLeftTop, mapSetting.positionLonRightButtom,  ImgNaturalWidth, ImgNaturalHeight)
    
    var xBalise = (newX + resTempo.x* (ImgWidth/ImgNaturalWidth) - baliseImg[i][0].width / 2) + "px"
    var yBalise = (newY  + resTempo.y* (ImgHeight/ImgNaturalHeight)- baliseImg[i][0].height / 2) + "px"
    
    if (xMov) baliseImg[i][0].style.left =  xBalise
    if (YMov) baliseImg[i][0].style.top = yBalise 

    i = i + 1
  });

  try {
    if (mapSetting.actiPosition) var a = true
  } catch (error) {
    mapSetting.actiPosition = true
  }

  console.log("mapSetting.actiPosition ", mapSetting.actiPosition)
  if (mapSetting.actiPosition) {
    navigator.geolocation.getCurrentPosition(success, error)
  } else {
    baliseUtili.style.display = "None"
  }

}

import zoomElem from "./ZoomElem.vue";


export default {
  components: {
    zoomElem
  },
  data() {
    return {
      press: false,
      x: 0,
      y: 0, 
      cadreElem : 0, 
      imageElem : 0,
      diffCoinUtiliX: 0,
      diffCoinUtiliY: 0,
      indiZoom: 100,
      sensibilitClicPosi: 100,
      widthOrigiImg: 0,
      heighthOrigiImg: 0,
      ratio: 1, 
      ImgNaturalWidth: 1,
      ImgNaturalHeight: 1,
      positionLatLeftTop: 0,
      positionLatRightButtom: 0,
      positionLonLeftTop: 0,
      positionLonRightButtom: 0,
      ListNomLieu: [],
      listLoca: [],
      baliseImg : [], 
      baliseUtili : 0,
      clicDownMouse : false
    }
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
    }
  },
  methods: {
    moveTel(e){

      let p = e.touches[0];

      var cdrWid = parseInt(this.cadreElem.style.width.substring(0, this.cadreElem.style.width.length - 2), 10)
      var cdrHei = parseInt(this.cadreElem.style.height.substring(0, this.cadreElem.style.height.length - 2), 10)
      var imgwid = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10)
      var imgHei = parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)

      var newX = p.clientX - this.diffCoinUtiliX; 
      var newY = p.clientY - this.diffCoinUtiliY;
      var newXBorder = p.clientX - this.diffCoinUtiliX + imgwid; 
      var newYBorder = p.clientY - this.diffCoinUtiliY + imgHei; 

      var xMov = false
      var yMov = false

      if ((newX <= 0 || newX < parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10)) && (newXBorder > cdrWid  || newX > parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10))) {this.imageElem.style.left = newX + "px"; xMov = true; }
      if ((newY <= 0 || newY < parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10)) && ((newYBorder > cdrHei) || newY > parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10))) {this.imageElem.style.top = newY + "px"; yMov = true;}
      baliseSurLaCarte (this.baliseImg, this.baliseUtili, newX, newY, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight, imgwid, imgHei, xMov, yMov)
    },
    
    move(e){
      
      if (this.clicDownMouse)
      {
        var mouseX = e.clientX - (window.innerWidth - 395)
        var mouseY = e.clientY - (window.innerHeight - 690)

        var cdrWid = parseInt(this.cadreElem.style.width.substring(0, this.cadreElem.style.width.length - 2), 10)
        var cdrHei = parseInt(this.cadreElem.style.height.substring(0, this.cadreElem.style.height.length - 2), 10)
        var imgwid = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10)
        var imgHei = parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)

        var newX = mouseX - this.diffCoinUtiliX; 
        var newY = mouseY - this.diffCoinUtiliY;
        var newXBorder = mouseX - this.diffCoinUtiliX + imgwid; 
        var newYBorder = mouseY - this.diffCoinUtiliY + imgHei; 

        var xMov = false
        var yMov = false

        if ((newX <= 0 || newX < parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10)) && (newXBorder > cdrWid  || newX > parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10))) {this.imageElem.style.left = newX + "px"; xMov = true; }
        if ((newY <= 0 || newY < parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10)) && ((newYBorder > cdrHei) || newY > parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10))) {this.imageElem.style.top = newY + "px"; yMov = true;}
        baliseSurLaCarte (this.baliseImg, this.baliseUtili, newX, newY, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight, imgwid, imgHei, xMov, yMov)
      }
    },
    
    initialClick(e) {

      if (window.innerWidth > 450) {

        this.clicDownMouse = true

        var mouseX = e.clientX - (window.innerWidth - 395)
        var mouseY = e.clientY - (window.innerHeight- 690)

        var borderLeft = parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10)
        var borderTop = parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10)
        
        var XClientOnMap = Math.round((mouseX - borderLeft - parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left.length - 2), 10))*(this.ImgNaturalWidth / parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width .length - 2), 10)))
        var YClientOnMap = Math.round((mouseY - borderTop - parseInt(this.imageElem.style.top .substring(0,  this.imageElem.style.top.length -  2), 10))*(this.ImgNaturalHeight /parseInt(this.imageElem.style.height.substring(0,this.imageElem.style.height .length - 2), 10)))

        var resLieuClic = verifClicButton (this.ListNomLieu, this.listLoca, XClientOnMap, YClientOnMap, this.sensibilitClicPosi, 
        this.mapSetting.positionLatLeftTop, this.mapSetting.positionLatRightButtom, this.mapSetting.positionLonLeftTop, this.mapSetting.positionLonRightButtom, 
        this.ImgNaturalWidth, this.ImgNaturalHeight)


        if (resLieuClic != "None")
        {
          this.$emit('sendSuggestion', resLieuClic)
        }

        var rect = this.imageElem.getBoundingClientRect();
        
        this.diffCoinUtiliX = mouseX - (rect.left  - (window.innerWidth - 395)) + parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10);
        this.diffCoinUtiliY = mouseY - (rect.top  - (window.innerHeight- 690)) + parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10);
        console.log(mouseX , mouseY, XClientOnMap, YClientOnMap, this.diffCoinUtiliX, this.diffCoinUtiliY, rect.left  - (window.innerWidth - 395), rect.top  - (window.innerHeight- 690), parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10), parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10));
      }
    }, 
    
    initialClickTel(e) {

      let p = e.touches[0];
      var borderLeft = parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10)
      var borderTop = parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10)
      
      var XClientOnMap = Math.round((p.clientX - borderLeft - parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left.length - 2), 10))*(this.ImgNaturalWidth / parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width .length - 2), 10)))
      var YClientOnMap = Math.round((p.clientY - borderTop - parseInt(this.imageElem.style.top .substring(0,  this.imageElem.style.top.length -  2), 10))*(this.ImgNaturalHeight /parseInt(this.imageElem.style.height.substring(0,this.imageElem.style.height .length - 2), 10)))

      var resLieuClic = verifClicButton (this.ListNomLieu, this.listLoca, XClientOnMap, YClientOnMap, this.sensibilitClicPosi, 
      this.mapSetting.positionLatLeftTop, this.mapSetting.positionLatRightButtom, this.mapSetting.positionLonLeftTop, this.mapSetting.positionLonRightButtom, 
      this.ImgNaturalWidth, this.ImgNaturalHeight)


      if (resLieuClic != "None")
      {
        this.$emit('sendSuggestion', resLieuClic)
      }


      var rect = this.imageElem.getBoundingClientRect();
      this.diffCoinUtiliX = p.clientX - rect.left + parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10);
      this.diffCoinUtiliY = p.clientY - rect.top + parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10);
      console.log(p.clientX , p.clientY, XClientOnMap, YClientOnMap, this.diffCoinUtiliX, this.diffCoinUtiliY, rect.left, rect.top, parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10), parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10));
    }, 
    endClick (image) {

      this.clicDownMouse = false

    },
    zoomCarte () { 

      var imgwid = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10)
      var imgHei = parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)
      var cdrWid = parseInt(this.cadreElem.style.width.substring(0, this.cadreElem.style.width.length - 2), 10)
      var cdrHei = parseInt(this.cadreElem.style.height.substring(0, this.cadreElem.style.height.length - 2), 10)
      var imgMargWei = parseInt(this.imageElem.style.left.substring(0, this.imageElem.style.left.length - 2), 10)
      var imgMargHei = parseInt(this.imageElem.style.top.substring(0, this.imageElem.style.top.length - 2), 10)


      //Agrandissement de l'image
      this.imageElem.style.width = imgwid  + this.indiZoom*this.ratio + "px";
      this.imageElem.style.height = imgHei  + this.indiZoom + "px"; 

      //Repositionnement au centre
      var imElLeft = Math.round(imgMargWei + valueZoomTranslat (imgwid, cdrWid, imgMargWei, this.indiZoom, this.ratio));
      var imElTop = Math.round(imgMargHei + valueZoomTranslat (imgHei, cdrHei, imgMargHei, this.indiZoom, 1));
      this.imageElem.style.left = imElLeft + "px";
      this.imageElem.style.top = imElTop + "px";

      baliseSurLaCarte (this.baliseImg, this.baliseUtili, imElLeft, imElTop, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight, imgwid+ this.indiZoom*this.ratio, imgHei+ this.indiZoom)


    },
    dezoomCarte () { 

      var imgwid = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10)
      var imgHei = parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)
      var cdrWid = parseInt(this.cadreElem.style.width.substring(0, this.cadreElem.style.width.length - 2), 10)
      var cdrHei = parseInt(this.cadreElem.style.height.substring(0, this.cadreElem.style.height.length - 2), 10)
      var imgMargWei = parseInt(this.imageElem.style.left.substring(0, this.imageElem.style.left.length - 2), 10)
      var imgMargHei = parseInt(this.imageElem.style.top.substring(0, this.imageElem.style.top.length - 2), 10)

      if (imgwid-this.indiZoom*this.ratio > cdrWid || imgHei-this.indiZoom*this.ratio > cdrHei)
      {
        //Agrandissement de l'image
        this.imageElem.style.width = imgwid  - this.indiZoom*this.ratio + "px";
        this.imageElem.style.height = imgHei  - this.indiZoom + "px"; 

        //Repositionnement au centre
        var imElLeft = Math.round(imgMargWei - valueZoomTranslat (imgwid, cdrWid, imgMargWei, this.indiZoom, this.ratio));
        var imElTop = Math.round(imgMargHei - valueZoomTranslat (imgHei, cdrHei, imgMargHei, this.indiZoom, 1));
        this.imageElem.style.left = imElLeft + "px";
        this.imageElem.style.top = imElTop + "px";

        baliseSurLaCarte (this.baliseImg, this.baliseUtili, imElLeft, imElTop, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight, imgwid - this.indiZoom*this.ratio, imgHei - this.indiZoom)
        
      } else {
        this.imageElem.style.width = cdrWid + "px";
        this.imageElem.style.height = cdrHei + "px";

        baliseSurLaCarte (this.baliseImg, this.baliseUtili, 0, 0, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight, cdrWid, cdrHei)
        
        this.imageElem.style.top = "0px";
        this.imageElem.style.left = "0px";

      }
    }

  }, mounted() {

    console.log("/var/lib/docker/overlay2/796e6ba7492640dd479df60807513a95bf89f22025d3d3e94e6012e60f9d00c2/diff/app/services/vivifone/node_modules/vue-beautiful-chat/src/MapSelec.vue");
    setTimeout(() => {

      var i = 0
      this.suggestions.forEach(elem => {
        this.ListNomLieu.push(elem.choice);
        this.listLoca.push([elem.localisationLat, elem.localisationLon]);
        
        this.baliseImg.push (this.$refs["baliseTest"+i])
        i = i + 1

      });

      this.baliseUtili = this.$refs["positionUtilisateur"]


      
      this.imageElem = this.$refs.imgMap;
      this.cadreElem = this.$refs.cadreForMapMenu;
      this.imageElem.setAttribute('draggable', false);

      var imageElemStyleWidth = this.mapSetting.sizeWidth
      var imageElemStyleHeight = this.mapSetting.sizeHeight
      this.ImgNaturalWidth = this.mapSetting.sizeWidth
      this.ImgNaturalHeight = this.mapSetting.sizeHeight

      this.indiZoom = this.mapSetting.zoomIndice //Math.round(imageElemStyleWidth/10);
      this.imageElem.style.width = imageElemStyleWidth+"px";
      this.imageElem.style.height = imageElemStyleHeight+"px";

      var tailleCadre = Math.round(window.innerWidth*0.9);
      if (tailleCadre > 450) tailleCadre = Math.round(368*0.9);
      
      if (window.innerWidth > 450) {
        this.cadreElem.style.left = "20px";
        this.cadreElem.style.top = "200px";
      } else {
        this.cadreElem.style.left = Math.round(window.innerWidth*0.05)+"px";
        this.cadreElem.style.top = (window.innerHeight-Math.round(tailleCadre/this.ratio)-50)+"px";
      }

      this.ratio = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10) / parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)

      this.cadreElem.style.width = tailleCadre+"px";
      this.cadreElem.style.height =  Math.round(tailleCadre/this.ratio)+"px";

      var imElLeft = (imageElemStyleWidth/2-Math.round(tailleCadre*this.ratio)/2);
      var imElTop = (imageElemStyleHeight/2-tailleCadre/2);
      this.imageElem.style.left = "-"+imElLeft+"px";
      this.imageElem.style.top = "-"+imElTop+"px";


      baliseSurLaCarte (this.baliseImg, this.baliseUtili, -1*imElLeft, -1*imElTop, this.listLoca, this.mapSetting, this.ImgNaturalWidth, this.ImgNaturalHeight)


      this.dezoom
      this.dezoom


    }, 4000)



  }
}

</script>

<style scoped>

#imgMap {
  position: absolute;
  z-index: 2;
  top : 0px;
  left : 0px;
}
.baliseTest {
  position: absolute;
  z-index: 3;
  top : 0px;
  left : 0px;
  width: 25px;
;
}
#cadreForMapMenu
{
  position: absolute;
  z-index: 1;
  border-radius: 30px;
  border: 2px #365ca7 solid;
  overflow: hidden;
}


.doubleButtonZoom {
    position: absolute;
    top: 80%;
    padding: 10px;
    /*background-color: #365ca7;*/
    border-radius: 10px;
    color: white;
    font-size: 18px;
}

#buttonDezoom {
    left: 60%;
    width: 80px;
    text-align: center;
}

#buttonZoom{
  left: 25%;
  width: 80px;
  text-align: center;

}

.imgZoom {
  width: 80px;
}

#backgroundFlou {
  position: absolute;
    top: 0px;
    left: 0px;
    background: white;
    width: 5000px;
    height: 1000px;
    opacity: 60%;
}

</style>

