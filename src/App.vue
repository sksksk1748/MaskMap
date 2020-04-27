<template>
  <div id="app">
    <div class="row no-gutters">
    <div class="col-sm-3">
      <div class="toolbox">
        <div class="sticky-top bg-white shadow-sm p-2">
          <div class="form-group d-flex">
            <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
            <div class="flex-fill">
              <!-- 建置地點選單 -->
              <select id="cityName" class="form-control"
              v-model="select.city" @change="select.area = ''"> <!-- 縣市選完後的變數會存入select裡的city-->
                <option value="">-- Select One --</option>
                <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                  {{ c.CityName }}
                </option>
              </select>
              <!-- 建置地點選單 -->
            </div>
          </div>
          <div class="form-group d-flex">
            <label for="area" class="mr-2 col-form-label text-right">地區</label>
            <div class="flex-fill">
              <select id="area" class="form-control" v-if="select.city.length"
                v-model="select.area" @change="updateSelect">
                <option value="">-- Select One --</option>
                <option :value="a.AreaName"
                  v-for="a in cityName.find((city) => city.CityName === select.city).AreaList"
                  :key="a.AreaName">
                  {{ a.AreaName }}
                </option>
              </select>
            </div>
          </div>
          <p class="mb-0 small text-muted text-right">請先選擇區域查看（藍色表示還有口罩，灰色表示無取得資料）</p>
        </div>

        <ul class="list-group">
          <template v-for="(item, key) in data">
             <a class="list-group-item text-left" :key="key"
              v-if="item.properties.county === select.city
                && item.properties.town === select.area"
              :class="{ 'highlight': item.properties.mask_adult || item.properties.mask_child}"
              @click="penTo(item)">
              <h3>{{ item.properties.name }}</h3>
              <p class="mb-0">
                成人口罩：{{ item.properties.mask_adult}} | 兒童口罩：{{ item.properties.mask_child}}
              </p>
              <p class="mb-0">地址：<a :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank" title="Google Map">
                {{ item.properties.address }}</a>
              </p>
            </a>
          </template>
        </ul>
      </div>
    </div>
    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';

let osmMap = {};

const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
};
const icons = {
  green: new L.Icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    ...iconsConfig,
  }),
  grey: new L.Icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
    ...iconsConfig,
  }),
};
const osm = {
  addMapMarker(x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<strong>${item.name}</strong> <br>
    口罩剩餘：<strong>成人 - ${item.mask_adult ? `${item.mask_adult} 個` : '未取得資料'}/ 兒童 - ${item.mask_child ? `${item.mask_child} 個` : '未取得資料'}</strong><br>
    地址: <a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a><br>
    電話: ${item.phone}<br>
    <small>最後更新時間: ${item.updated}</small>`);
  },
  removeMapMarker() {
    osmMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        osmMap.removeLayer(layer);
      }
    });
  },
  penTo(x, y, item) {
    console.log(x);
    console.log(y);
    console.log(item);
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    osmMap.panTo(new L.LatLng(y, x));
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<strong>${item.name}</strong> <br>
    口罩剩餘：<strong>成人 - ${item.mask_adult ? `${item.mask_adult} 個` : '未取得資料'}/ 兒童 - ${item.mask_child ? `${item.mask_child} 個` : '未取得資料'}</strong><br>
    地址: <a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a><br>
    電話: ${item.phone}<br>
    <small>最後更新時間: ${item.updated}</small>`).openPopup();
  },
};

export default {
  name: 'App',
  data: () => ({
    data: {},
    cityName,
    osmMap: {},
    select: { //  for縣市選單
      city: '臺北市', // 預設縣市選單為台北市
      area: '大安區', // 預設區域設為大安區
    },
  }),
  /* components: {
  }, */
  // components 更改為 methods
  methods: {
    updateMarker() {
      const pharmacies = this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city;
        }
        return pharmacy.properties.town === this.select.area;
      });
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy; // 新寫法(結構式寫法)
        osm.addMapMarker(
          // pharmacy.geometry.coordinates[1], // 如果程式碼太長，可以改結構式寫法
          // pharmacy.geometry.coordinates[0], // 如果程式碼太長，可以改結構式寫法
          geometry.coordinates[0], // 新寫法(結構式寫法)
          geometry.coordinates[1], // 新寫法(結構式寫法)
          properties,
        );
      });
      this.penTo(pharmacies[0]);
    },
    removeMarker() {
      osm.removeMapMarker();
    },
    penTo(pharmacy) {
      const { properties, geometry } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
    updateSelect() {
      this.removeMarker();
      this.updateMarker();
      const pharmacy = this.data.find((item) => (
        item.properties.town === this.select.area));
      const { geometry, properties } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
  },
  mounted() {
    osmMap = L.map('map', { //  'map'對應到<div id="map">
      center: [25.03, 121.55], // 地圖起始座標
      zoom: 15, //  地圖縮放
    });
    //  補上圖磚，地圖內容
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
      maxZoom: 18,
    }).addTo(osmMap);
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json'; // 藥局資料
    this.$http.get(url).then((response) => {
      this.data = response.data.features;
      this.updateMarker(); // 取得遠端資料後，插入圖標
    });
    // 插入圖標
    // L.marker([25.03, 121.55]).addTo(osmMap);
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
}
.home {
  position: relative;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
.list-group-item{
  background-color: rgb(65, 178, 243);
}
</style>
