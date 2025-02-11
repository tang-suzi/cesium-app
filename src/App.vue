<template>
  <div id="cesiumContainer"></div>
</template>

<script setup>
import * as Cesium from "cesium";
import { onMounted } from "vue";
onMounted(async () => {
  console.log(Cesium)
  // const viewer = new Cesium.Viewer("cesiumContainer");
  const custom = new Cesium.ArcGisMapServerImageryProvider({
    url: "https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer",
  });

  const viewer = new Cesium.Viewer("cesiumContainer", {
    baseLayerPicker: false, // 地图切换
    // imageryProvider: custom, // 影像图层,设置图像提供的程序
    terrainProvider: await Cesium.createWorldTerrainAsync({
      requestWaterMask: true, // 水面
      requestVertexNormals: true, // 法线
    }), // 设置地形
    geocoder: false, // 地名搜索
    homeButton: false, // 回到初始位置
    sceneModePicker: false, // 三维/二维切换
    navigationHelpButton: false, // 帮助按钮
    animation: false, // 时间轴
    timeline: false, // 时间轴
    fullscreenButton: false, // 全屏按钮
    vrButton: false, // VR按钮
    infoBox: false, // 信息框
    // selectionIndicator: false, // 选择指示器
    // navigationInstructionsInitiallyVisible: false, // 初始导航指示
  });

  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(-77.05597, 38.87126, 2000), // 设置位置经纬度和高度
    duration: 0, // 动画持续时间
    orientation: {
      heading: Cesium.Math.toRadians(90), // 方向
      pitch: Cesium.Math.toRadians(-90), // 俯仰 0为水平视角
      roll: 0, // 翻滚
    },
  });
});
</script>

<style scoped>
html,
body,
#cesiumContainer {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  position: relative;
}
</style>
