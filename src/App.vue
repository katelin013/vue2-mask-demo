<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="flex col-sm-12 p-2 bg-white">
        <!-- 選擇縣市 -->
        <div class="form-inline">
          <label for="cityName" class="col-form-label mr-2 text-right">縣市</label>
          <div class="mr-3">
            <!-- 這裏 v-model 會先綁定 select 中的 city, 目的是為了找到對應的 city, 所以不能為空-->
            <select id="cityName" class="form-control" v-model="select.city">
              <option value="">請選擇縣市</option>
              <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                {{ c.CityName }}
              </option>
            </select>
          </div>
          <!-- 選擇地區 -->
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div>
            <select id="area" class="form-control" v-model="select.area">
              <option value="">請選擇地區</option>
              <!-- v-for find 會找到 cityName 是第一個 Select 所選到的選項所對應的 AreaList-->
              <option :value="a.AreaName"
                      v-for="a in cityName.find((city) => city.CityName === select.city).AreaList"
                      :key="a.AreaName">
                {{ a.AreaName }}
              </option>
            </select>
          </div>
        </div>

      </div>
    </div>

    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>

import cityName from "./assets/cityName.json";
import leaflet from 'leaflet';

let openStreetMap = {};

export default {
  name: 'App',
  components: {

  },
  data: () => ({
    data:[],
    cityName,
    latitude : 25.047445,
    longitude: 121.513975,
    select: {
      city: '臺北市',
      area: ''
    },
  }),
  mounted() {
    // 取得各醫院診所的資訊
    const api = "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
    this.$http.get(api)
    .then((response) => {
      this.data = response.data.features
    })

    // 設定預設位置
    openStreetMap = leaflet.map('map').setView([this.latitude, this.longitude], 19);

    // tileLayer 方法主要是對疊加在地圖上的操作進行設定
    leaflet.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '<a href="https://www.openstreetmap.org/">OSM</a>',
      maxZoom :19,
      minZoom :15,
    }).addTo(openStreetMap);
  },
  computed: {
    pharmacies(){
      // 透過 filter 找到相對應的縣市/區域
      return this.data.filter((pharamacy) => {
        if(!this.select.area){
          return pharamacy.properties.county === this.select.city;
        }
        return pharamacy.properties.town === this.select.area;
      })
    }
  },
  watch: {
    // 監聽 pharmacies 的變化
    pharmacies(){
      this.updateMap();
    },
    'select.area':function(){
      // openStreetMap.panTo([this.latitude, this.longitude]).setZoom(17)
    }
  },
  methods: {
    updateMap(){
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof leaflet.Marker){
          openStreetMap.removeLayer(layer);
        }
      })

      this.pharmacies.forEach((pharmacy) => {
        console.log(pharmacy)
        leaflet.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0],
        ]).addTo(openStreetMap).bindPopup(`
          <p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
          地址: ${pharmacy.properties.address}<br>
          電話: ${pharmacy.properties.phone}<br>
          <small>最後更新時間: ${pharmacy.properties.updated}</small>
          `);
      });
    },
  }
}
</script>

<style lang="scss">
  @import 'bootstrap/scss/bootstrap';

  #map {
    position: relative;
    height: 100vh;
  }
</style>
