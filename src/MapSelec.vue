<template>
  <div>
    <div id="cadreForMapMenu" ref="cadreForMapMenu" @mousedown="initialClick" @mousemove.passive="move" @mouseup="endClick" @touchstart="initialClickTel" @touchmove.passive="moveTel">
      <img id="imgMap" ref="imgMap" :src="'/api/assets/test_bot/'+mapSetting.imgMap" > 
    </div>
    <div @mousedown="zoomCarte" class="doubleButtonZoom" id="buttonZoom"> 
      +
    </div>
    <div @mousedown="dezoomCarte" id="buttonDezoom" class="doubleButtonZoom">
      -
    </div>
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
  
  console.log(xClient, yClient, locPix[0])

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


export default {
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
      mouseIsDown: false
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

      console.log(newX, p.clientX, this.diffCoinUtiliX)


      if ((newX <= 0 || newX < parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10)) && (newXBorder > cdrWid  || newX > parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10))) this.imageElem.style.left = newX + "px";
      if ((newY <= 0 || newY < parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10)) && ((newYBorder > cdrHei) || newY > parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10))) this.imageElem.style.top = newY + "px";
    },
    
    move(e){

      if (this.mouseIsDown)
      {
        var clientX = e.clientX - (window.innerWidth - 370 - 25)
        var clientY = e.clientY - (window.innerHeight - 590 - 100)


        var cdrWid = parseInt(this.cadreElem.style.width.substring(0, this.cadreElem.style.width.length - 2), 10)
        var cdrHei = parseInt(this.cadreElem.style.height.substring(0, this.cadreElem.style.height.length - 2), 10)
        var imgwid = parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width.length - 2), 10)
        var imgHei = parseInt(this.imageElem.style.height.substring(0, this.imageElem.style.height.length - 2), 10)

        var newX = clientX - this.diffCoinUtiliX; 
        var newY = clientY - this.diffCoinUtiliY;
        var newXBorder = clientX - this.diffCoinUtiliX + imgwid; 
        var newYBorder = clientY - this.diffCoinUtiliY + imgHei; 


        if ((newX <= 0 || newX < parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10)) && (newXBorder > cdrWid  || newX > parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left .length - 2), 10))) this.imageElem.style.left = newX + "px";
        if ((newY <= 0 || newY < parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10)) && ((newYBorder > cdrHei) || newY > parseInt(this.imageElem.style.top .substring(0, this.imageElem.style.top .length - 2), 10))) this.imageElem.style.top = newY + "px";
      }

    },
    
    initialClick(e) {

      console.log("beginClick")


      if (window.innerWidth > 450) {

        var clientX = e.clientX - (window.innerWidth - 370 - 25)
        var clientY = e.clientY - (window.innerHeight - 590 - 100)

        var borderLeft = parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10)
        var borderTop = parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10)
        
        var XClientOnMap = Math.round((clientX - borderLeft - parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left.length - 2), 10))*(this.ImgNaturalWidth / parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width .length - 2), 10)))
        var YClientOnMap = Math.round((clientY - borderTop - parseInt(this.imageElem.style.top .substring(0,  this.imageElem.style.top.length -  2), 10))*(this.ImgNaturalHeight /parseInt(this.imageElem.style.height.substring(0,this.imageElem.style.height .length - 2), 10)))

        console.log(clientX)

        var resLieuClic = verifClicButton (this.ListNomLieu, this.listLoca, XClientOnMap, YClientOnMap, this.sensibilitClicPosi, 
        this.mapSetting.positionLatLeftTop, this.mapSetting.positionLatRightButtom, this.mapSetting.positionLonLeftTop, this.mapSetting.positionLonRightButtom, 
        this.ImgNaturalWidth, this.ImgNaturalHeight)


        if (resLieuClic != "None")
        {
          this.$emit('sendSuggestion', resLieuClic)
        }


        var rect = this.imageElem.getBoundingClientRect();
        this.diffCoinUtiliX = clientX - (rect.left - (window.innerWidth - 370 + 40)) + parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10);
        this.diffCoinUtiliY = clientY - (rect.top - (window.innerHeight - 590 - 100)) + parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10);

        this.mouseIsDown = true;
      }
    }, 
    
    initialClickTel(e) {

      let p = e.touches[0];
      var borderLeft = parseInt(this.cadreElem.style.left .substring(0,  this.cadreElem.style.left.length -  2), 10)
      var borderTop = parseInt(this.cadreElem.style.top .substring(0,  this.cadreElem.style.top.length -  2), 10)
      
      var XClientOnMap = Math.round((p.clientX - borderLeft - parseInt(this.imageElem.style.left .substring(0, this.imageElem.style.left.length - 2), 10))*(this.ImgNaturalWidth / parseInt(this.imageElem.style.width.substring(0, this.imageElem.style.width .length - 2), 10)))
      var YClientOnMap = Math.round((p.clientY - borderTop - parseInt(this.imageElem.style.top .substring(0,  this.imageElem.style.top.length -  2), 10))*(this.ImgNaturalHeight /parseInt(this.imageElem.style.height.substring(0,this.imageElem.style.height .length - 2), 10)))

      console.log(p.clientX, borderLeft, this.imageElem.style.left, this.imageElem.style.width)


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

    }, 
    endClick (image) {

      this.mouseIsDown = false;

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
      this.imageElem.style.left = Math.round(imgMargWei + valueZoomTranslat (imgwid, cdrWid, imgMargWei, this.indiZoom, this.ratio)) + "px";
      this.imageElem.style.top = Math.round(imgMargHei + valueZoomTranslat (imgHei, cdrHei, imgMargHei, this.indiZoom, 1))+ "px";

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
        this.imageElem.style.left = Math.round(imgMargWei - valueZoomTranslat (imgwid, cdrWid, imgMargWei, this.indiZoom, this.ratio)) + "px";
        this.imageElem.style.top = Math.round(imgMargHei - valueZoomTranslat (imgHei, cdrHei, imgMargHei, this.indiZoom, 1))+ "px";
      } else {
        this.imageElem.style.width = cdrWid + "px";
        this.imageElem.style.height = cdrHei + "px";
        this.imageElem.style.top = "0px";
        this.imageElem.style.left = "0px";
      }
    }

  }, mounted() {

    
    setTimeout(() => {


      this.suggestions.forEach(elem => {
        this.ListNomLieu.push(elem.choice);
        this.listLoca.push([elem.localisationLat, elem.localisationLon]);
      });
      
      this.imageElem = this.$refs.imgMap;
      this.cadreElem = this.$refs.cadreForMapMenu;
      this.imageElem.setAttribute('draggable', false);

      var imageElemStyleWidth = this.mapSetting.sizeWidth
      var imageElemStyleHeight = this.mapSetting.sizeheight
      this.ImgNaturalWidth = 2317
      this.ImgNaturalHeight = 1548


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


      this.imageElem.style.left = "-"+(imageElemStyleWidth/2-Math.round(tailleCadre*this.ratio)/2)+"px";
      this.imageElem.style.top = "-"+(imageElemStyleHeight/2-tailleCadre/2)+"px";

      this.dezoom
      this.dezoom


    }, 200)



  }
}

</script>

<style>
#imgMap {
  position: absolute;
  top : 0px;
  left : 0px;
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
    background-color: #365ca7;
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
</style>
