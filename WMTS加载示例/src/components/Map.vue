<template>
  <div id="map" class="map"></div>
</template>

<script setup>
import Map from "ol/Map";
import OSM from "ol/source/OSM";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import WMTS from "ol/source/WMTS";

import TileGrid from "ol/tilegrid/TileGrid";
import XYZ from "ol/source/XYZ";
import TileArcGISRest from "ol/source/TileArcGISRest";
import WMTSTileGrid from "ol/tilegrid/WMTS";
import { get as getProjection } from "ol/proj";
import { getTopLeft, getWidth } from "ol/extent";
import { onMounted } from "vue";
import proj4 from "proj4";
import { register } from "ol/proj/proj4";
import { Projection, addProjection } from "ol/proj";

function initMap() {
  proj4.defs(
    "EPSG:4490",
    'GEOGCS["China Geodetic Coordinate System 2000",DATUM["China_2000",SPHEROID["CGCS2000",6378137,298.257222101,AUTHORITY["EPSG","1024"]],AUTHORITY["EPSG","1043"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4490"]]'
  );
  register(proj4);

  //重写projection4490，
  const projection = new Projection({
    code: "EPSG:4490",
    units: "degrees",
    axisOrientation: "neu",
  });
  projection.setExtent([-180, -90, 180, 90]);
  projection.setWorldExtent([-180, -90, 180, 90]);
  addProjection(projection);

  //const projection = getProjection('EPSG:4490');
  const projectionExtent = projection.getExtent();
  const size = getWidth(projectionExtent) / 256;
  const resolutions = new Array(23);
  const matrixIds = new Array(23);
  for (let z = 0; z < 23; ++z) {
    // generate resolutions and matrixIds arrays for this WMTS
    resolutions[z] = size / Math.pow(2, z + 1);
    matrixIds[z] = z;
  }
  let _tileInfo = {
    tileSize: 256,
    origin: [-180.0, 90],
    extent: [
      120.65556572149626, 31.05055834136383, 121.3055914245038,
      31.600791366022317,
    ],
    resolutions: [
      0.703125, 0.3515625, 0.17578125, 0.087890625, 0.0439453125, 0.02197265625,
      0.010986328125, 0.0054931640625, 0.00274658203125, 0.001373291015625,
      6.866455078125e-4, 3.4332275390625e-4, 1.71661376953125e-4,
      8.58306884765625e-5, 4.291534423828125e-5, 2.1457672119140625e-5,
      1.0728836059570312e-5, 5.364418029785156e-6, 2.682209014892578e-6,
      1.341104507446289e-6, 6.705522537231445e-7, 3.3527612686157227e-7,
      1.6763806343078613e-7,
    ],
  };
  console.log(resolutions);
  console.log(_tileInfo.resolutions);
  console.log(resolutions.length, _tileInfo.resolutions.length);

  let arcGISLayers = new TileLayer({
    source: new XYZ({
      url: `http://180.97.207.58:8082/gis_map/sicp/ks_csdt_0428/MapServer/tile/{z}/{y}/{x}`,
      tileGrid: new TileGrid(_tileInfo),
      projection: projection,
    }),
  });

  const wmtsTileGrid = new WMTSTileGrid({
    origin: getTopLeft(projectionExtent),
    resolutions: resolutions,
    matrixIds: matrixIds,
  });

  const map = new Map({
    layers: [
      //   new TileLayer({
      //     source: new OSM(),
      //   }),
      arcGISLayers,
      new TileLayer({
        opacity: 0.7,
        source: new WMTS({
          url: "http://180.97.207.58:8082/gis_map/ccf7e74739f84b2d8a1fe88970e3d607/sicp/KSBlueMap2020_CGCS2000/Tile/WMTS?",
          layer: "BaseData_KSBlueMap2020_CGCS2000", // 图层名
          version: "1.0.0", // WMTS版本
          // 投影坐标系矩阵集，一定要和WMTS capabilities文档中一致，否则会加载失败
          matrixSet: "default028mm",
          format: "image/png", // 图片格式
          projection: projection, // 投影坐标系
          requestEncoding: "KVP", // 请求的编码方式，默认就是'KVP'
          // 在WMTS capabilities文档中对应的样式名，一定要写，否则会加载失败
          style: "default",
          tileGrid: wmtsTileGrid,
        }),
      }),
    ],
    target: "map",
    view: new View({
      projection: projection,
      center: [121.05, 31.38],
      zoom: 11,
    }),
  });
  console.log(map);
}

// 生命周期钩子
onMounted(() => {
  initMap();
});
</script>

<style scope>
#map {
  width: 100%;
  height: 1500px;
}
</style>
