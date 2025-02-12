<template>
  <div id="cesiumContainer"></div>
</template>

<script setup>
import * as Cesium from "cesium";
import { onMounted } from "vue";
import geojson from "./assets/geo/area.js";
onMounted(async () => {
  console.log(Cesium);
  // const viewer = new Cesium.Viewer("cesiumContainer");
  const custom = new Cesium.ArcGisMapServerImageryProvider({
    url: "//services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer",
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

  // viewer.camera.flyTo({
  //   destination: Cesium.Cartesian3.fromDegrees(-77.05597, 38.87126, 2000), // 设置位置经纬度和高度
  //   duration: 0, // 动画持续时间
  //   orientation: {
  //     heading: Cesium.Math.toRadians(90), // 方向
  //     pitch: Cesium.Math.toRadians(-90), // 俯仰 0为水平视角
  //     roll: 0, // 翻滚
  //   },
  // });
  viewer.camera.flyTo({
    destination: new Cesium.Cartesian3(1332761, -4662399, 4137888),
    orientation: {
      heading: 0.6,
      pitch: -0.66,
    },
  });

  const heightStyle = new Cesium.Cesium3DTileStyle({
    color: {
      conditions: [
        ["${Height} >= 300", "rgba(45, 0, 75, 0.5)"],
        ["${Height} >= 200", "rgb(102, 71, 151)"],
        ["${Height} >= 100", "rgb(170, 162, 204)"],
        ["${Height} >= 50", "rgb(224, 226, 238)"],
        ["${Height} >= 25", "rgb(252, 230, 200)"],
        ["${Height} >= 10", "rgb(248, 176, 87)"],
        ["${Height} >= 5", "rgb(198, 106, 11)"],
        ["true", "rgb(127, 59, 8)"],
      ],
    },
  });
  const tileset = await Cesium.Cesium3DTileset.fromIonAssetId(75343);
  tileset.style = heightStyle;
  viewer.scene.primitives.add(tileset);

  const neighborhoodsPromise = await Cesium.GeoJsonDataSource.load(geojson);
  viewer.dataSources.add(neighborhoodsPromise);
  let neighborhoods = neighborhoodsPromise.entities;
  for (let index = 0; index < neighborhoods.values.length; index++) {
    const entity = neighborhoods.values[index];
    if (Cesium.defined(entity.polygon)) {
      entity.name = entity.properties.boro_name;
      entity.polygon.material = Cesium.Color.fromRandom({
        red: 0.1,
        maximumGreen: 0.5,
        minimumBlue: 0.5,
        alpha: 0.6,
      });

      entity.polygon.classificationType = Cesium.ClassificationType.TERRAIN;

      const polyPositions = entity.polygon.hierarchy.getValue(
        Cesium.JulianDate.now()
      ).positions;
      let polyCenter = Cesium.BoundingSphere.fromPoints(polyPositions).center;
      polyCenter = Cesium.Ellipsoid.WGS84.scaleToGeodeticSurface(polyCenter);
      entity.position = polyCenter;

      // 生成标签
      entity.label = {
        text: entity.name, // 标签内容
        showBackground: true, // 显示背景
        // font: "14px monospace", // 字体
        scale: 0.6, // 缩放
        horizontalOrigin: Cesium.HorizontalOrigin.CENTER, // 水平位置
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM, // 垂直位置
        pixelOffset: new Cesium.Cartesian2(0, -9), // 偏移
        eyeOffset: new Cesium.Cartesian3(0, 0, -5), // 眼睛偏移
        distanceDisplayCondition: new Cesium.DistanceDisplayCondition(
          10.0,
          8000.0
        ), // 显示距离
        disableDepthTestDistance: 100, // 深度测试
      };
    }
  }

  const kmlOptions = {
    camera: viewer.scene.camera,
    canvas: viewer.scene.canvas,
    clampToGround: true,
  };

  // http://catalog.opendata.city/dataset/pediacities-nyc-neighborhoods/resource/9177848-3c58-449c-a3f9-365ed2d6e4d2

  const geocachePromise = await Cesium.KmlDataSource.load(
    "./src/SampleData/sampleGeocacheLocations.kml",
    kmlOptions
  );
  viewer.dataSources.add(geocachePromise);

  const geocacheEntities = geocachePromise.entities.values;

  for (let i = 0; i < geocacheEntities.length; i++) {
    const entity = geocacheEntities[i];
    console.log(entity)
    console.log(Cesium.defined(entity.billboard))
    if (Cesium.defined(entity.billboard)) {
      entity.billboard.verticalOrigin = Cesium.VerticalOrigin.BOTTOM; // 垂直位置
      // entity.billboard.image = "./assets/vue.svg"; // 图片
      entity.label = undefined; // 标签禁用
      entity.billboard.distanceDisplayCondition =
        new Cesium.DistanceDisplayCondition(10.0, 20000.0); // 显示距离
      const cartographicPosition = Cesium.Cartographic.fromCartesian(
        entity.position.getValue(Cesium.JulianDate.now())
      ); // 笛卡尔坐标转经纬度
      const latitude = Cesium.Math.toDegrees(cartographicPosition.latitude); // 纬度
      const longitude = Cesium.Math.toDegrees(cartographicPosition.longitude); // 经度
      const description = `<table class="cesium-infoBox-defaultTable cesium-infoBox-defaultTable-lighter">
          <tbody>
            <tr>
              <th>Longitude</th>
              <td>${longitude.toFixed(5)}</td>
            </tr>
            <tr>
              <th>Latitude</th>
              <td>${latitude.toFixed(5)}</td>
            </tr>
            <tr>
              <th>实时人流</th>
              <td>${Math.floor(Math.random()*20000)}</td>
            </tr>
            <tr>
              <th>安全等级</th>
              <td>${Math.floor(Math.random()*5)}</td>
            </tr>
          </tbody>
        </table>`;
        entity.description = description;
    }
  }

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
