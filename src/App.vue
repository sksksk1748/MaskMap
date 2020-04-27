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
              <select id="area" class="form-control">
                <option value="">-- Select One --</option>
              </select>
            </div>
          </div>
          <p class="mb-0 small text-muted text-right">請先選擇區域查看（綠色表示還有口罩）</p>
        </div>

        <ul class="list-group">
          <template>
            <a class="list-group-item text-left">
              <h3>藥局名稱</h3>
              <p class="mb-0">
                成人口罩：1 | 兒童口罩：2
              </p>
              <p class="mb-0">地址：<a href="https://www.google.com.tw/maps/place/..."
                target="_blank" title="Google Map">
                地址</a>
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

console.log(L);

export default {
  name: 'App',
  data: () => ({
    data: [],
    cityName,
    select: { //  for縣市選單
      city: '臺北市', // 預設縣市選單為台北市
    },
  }),
  /* components: {
  }, */
  // components 更改為 methods
  methods: {
    updateMap() {
      const pharmaies = this.data.filter((pharmacy) => (
        pharmacy.properties.county === this.select.city));

      pharmaies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy; // 新寫法(結構式寫法)
        L.marker([
          // pharmacy.geometry.coordinates[1], // 如果程式碼太長，可以改結構式寫法
          // pharmacy.geometry.coordinates[0], // 如果程式碼太長，可以改結構式寫法
          geometry.coordinates[1], // 新寫法(結構式寫法)
          geometry.coordinates[0], // 新寫法(結構式寫法)
        ]).addTo(osmMap).bindPopup(`<strong>${properties.name}</strong> <br>
    口罩剩餘：<strong>成人 - ${properties.mask_adult ? `${properties.mask_adult} 個` : '未取得資料'}/ 兒童 - ${properties.mask_child ? `${properties.mask_child} 個` : '未取得資料'}</strong><br>
    地址: <a href="https://www.google.com.tw/maps/place/${properties.address}" target="_blank">${properties.address}</a><br>
    電話: ${properties.phone}<br>
    <small>最後更新時間: ${properties.updated}</small>`);
      });
    },
  },
  mounted() {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json'; // 藥局資料
    this.$http.get(url).then((response) => {
      this.data = response.data.features;
      this.updateMap(); // 取得遠端資料後，插入圖標
    });
    osmMap = L.map('map', { //  'map'對應到<div id="map">
      center: [25.03, 121.55], // 地圖起始座標
      zoom: 15, //  地圖縮放
    });
    //  補上圖磚，地圖內容
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
      maxZoom: 18,
    }).addTo(osmMap);
    // 插入圖標
    L.marker([25.03, 121.55]).addTo(osmMap);
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
