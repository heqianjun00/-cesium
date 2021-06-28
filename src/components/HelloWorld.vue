<style scoped>
.container {
  position: relative;
}
.control-panel {
  width: 500px;
  height: 200px;
  z-index: 2;
  position: absolute;
  background-color: rgba(0,0,0,.3);
  left: 100px;
  top: 50px;
}
.slider1, .slider2 {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 10px;
  margin-bottom: 10px;
}
.el-slider1 {
  width: 300px;
}
.demonstration {
  margin-right: 15px;
  color: #fff;
}
.parameter {
  margin-left: 10px;
  color: #fff;
}
</style>
<template>
  <div class="container">
      <div class="control-panel">
        <div class="sliders">
          <div class="slider1">
            <span class="demonstration">视角水平张角</span>
            <el-slider v-model="horizontalViewAngle" class="el-slider1" :max="120" :min="1"></el-slider>
            <span class="parameter">{{horizontalViewAngle}}</span>
          </div>
          <div class="slider2">
            <span class="demonstration">视角垂直张角</span>
            <el-slider v-model="verticalViewAngle" class="el-slider1" :max="90" :min="1"></el-slider>
            <span class="parameter">{{verticalViewAngle}}</span>
          </div>
        </div>
        <div class="btns">
          <el-button type="success" @click="addVisible">添加可视域</el-button>
          <el-button type="warning" @click="removeOneVisible">逐个清除</el-button>
          <el-button type="danger" @click="removeAllVisible">全部清除</el-button>
        </div>
      </div>
      <div id="cesiumContainer">
      </div>
  </div>
</template>

<script>
import ViewShed from '../../static/js/ViewShed.js';
let viewer, handler
export default
{
  data()
   {
    return {
      viewShed: {},
      startPosition: null,      //起始坐标
      endPosition: null,        //终点坐标
      viewShedArr: [],          //存储可视域区域的数组
      horizontalViewAngle: 90,  //视角水平张角
      verticalViewAngle: 60,    //视角垂直张角
     }
   },
  components:{},
  created()
   {

   },
  mounted()
   {
     this.initMap()
   },
  methods:
   {
     
     /**
      * @method 初始化地球
      */
     initMap()
     {
       viewer = new Cesium.Viewer('cesiumContainer',{
         terrainProvider: Cesium.createWorldTerrain(),
         // 取消地图上的控件
         geocoder: false,
         homeButton: false,
         sceneModePicker: false,
         baseLayerPicker: false,
         navigationHelpButton: false,
         animation: false,
         shouldAnimate: false,
         timeline: false,
         fullscreenButton: false
       })
       window.viewer = viewer
       viewer.scene.debugShowFramesPerSecond = true  //显示帧数
       // 下面这个是在cesium官网沙盒案例中找的3D模型
       var tileset = new Cesium.Cesium3DTileset({
          url: Cesium.IonResource.fromAssetId(40866),
        })
        viewer.scene.primitives.add(tileset)
        viewer.zoomTo(tileset)
     },

     /**
      * @method 添加可视区域
      */
     addVisible()
     {
      let i = 0
      handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas)
      handler.setInputAction(movement =>
      {
        i++
        if(i === 1)
        {
          this.startPosition = viewer.scene.pickPosition(movement.position) //鼠标点击一次获取开始坐标
          if(!this.startPosition) return
          this.viewShed = new ViewShed(viewer,{
            viewPosition: this.startPosition,
            viewPositionEnd: this.startPosition,
            horizontalViewAngle: this.horizontalViewAngle,
            verticalViewAngle: this.verticalViewAngle
          })
          handler.setInputAction(movement => //鼠标移动的事件
            {
              this.endPosition =  viewer.scene.pickPosition(movement.endPosition)
              if(!this.endPosition) return
              this.viewShed.updatePosition(this.endPosition)
              if(!this.viewShed.sketch) {
                this.viewShed.drawSketch()
              }
            }, Cesium.ScreenSpaceEventType.MOUSE_MOVE)         
        }
        if(i === 2) //鼠标点击两次获取结束坐标
        { 
          i = 0
          this.endPosition =  viewer.scene.pickPosition(movement.position)
          this.viewShed.updatePosition(this.endPosition)
          this.viewShed.update()
          handler = handler && handler.destroy()  //销毁鼠标事件
          this.viewShedArr.push(this.viewShed)
        } 
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK)
     },

     /**
      * @method 逐个去除可视区域
      */
     removeOneVisible()
     {
       
       let i = this.viewShedArr.length
       if(i === 0 ) return
       this.viewShedArr[i-1].clear()
       i--
       this.viewShedArr.pop()
     },

     /**
      * @method 一次去除所有可视区域
      */
     removeAllVisible()
     {
       if(this.viewShedArr === null) return
       for(let i=0; i<this.viewShedArr.length; i++){
         this.viewShedArr[i].clear()
       }
     }
   },
}
</script>

