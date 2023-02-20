<template>
  <div id="app">
    <div class="menus-area">
      <!-- 選單 -->
      <h3>新竹市違規自動照相桿地圖</h3>
      <div class="menus">
        <div class="menu-group">
          <label for="area" class="menu-text">地區</label>
          <select
            id="area"
            class="form-select"
            v-model="select.area"
            aria-label="選擇地區"
            style="display: inline; width: unset; margin-left: 8px"
          >
            <option value="" selected>請選擇地區</option>
            <option value="新竹市北區">北區</option>
            <option value="新竹市東區">東區</option>
            <option value="新竹市香山區">香山區</option>
          </select>
        </div>
        <div class="menu-group">
          <label for="road" class="menu-text">街道</label>
          <select
            id="road"
            class="form-select"
            v-model="select.road"
            aria-label="選擇街道"
            style="display: inline; width: unset; margin-left: 8px"
          >
            <option value="" selected>請選擇街道</option>
            <option
              :value="city.road"
              v-for="(city, index) in roadFilter.filter(
                (area) => area.site === select.area
              )"
              :key="index"
            >
              {{ city.road }}
            </option>
          </select>
        </div>
        <div class="menu-group">
          <div class="form-check form-switch" style="padding-left: 0">
            <label class="form-check-label" for="techSwitch">科技執法</label>
            <input
              v-model="techSwitchOn"
              class="form-check-input"
              type="checkbox"
              role="switch"
              id="techSwitch"
              style="margin-left: 5px; float: right"
            />
          </div>
        </div>
        <div class="menu-group" :value="speedMeasuringApps">
          <p>
            共 <span>{{ speedMeasuringApps.length }}</span> 筆結果
          </p>
        </div>
      </div>
      <!-- 地圖 -->
      <div class="map-area">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>
<script>
// 導入內部檔案
import HsinchuCountry from "./assets/HsinchuCountry.json";
import Leaflet from "leaflet";

let openStreetMap = {};

export default {
  name: "App",
  data: () => ({
    HsinchuCountry,
    data: [],
    select: {
      area: "",
      road: "",
    },
    techSwitchOn: false,
  }),
  components: {},
  computed: {
    speedMeasuringApps() {
      return this.data.filter((app) => {
        if (!this.select.area) {
          if (this.techSwitchOn === false) {
            return app.地點.includes(this.select.area);
          } else {
            return (
              app.地點.includes(this.select.area) &&
              app.備註.includes("科技執法")
            );
          }
        } else {
          if (this.techSwitchOn === false) {
            return app.地點.includes(this.select.road);
          } else {
            return (
              app.地點.includes(this.select.road) &&
              app.備註.includes("科技執法")
            );
          }
        }
      });
    },
    roadFilter() {
      return this.HsinchuCountry.filter((country) => {
        let roadInData = [];
        this.data.forEach((item) => roadInData.push(item.地點));
        if (
          roadInData.find((element) => element.includes(country.road)) !==
          undefined
        ) {
          return country.road;
        }
      });
    },
  },
  methods: {
    updateMap() {
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof Leaflet.Marker) {
          openStreetMap.removeLayer(layer);
        }
      });

      this.speedMeasuringApps.forEach((app) => {
        Leaflet.marker([app.緯度, app.經度]).addTo(openStreetMap)
          .bindPopup(`<p><strong style="font-size: 20px;">地點：${
          app.地點
        }</strong></p>
          <strong style="font-size: 18px; color: #d45345;">速限：${
            app.速限 ? app.速限 : "無"
          }</strong><br>
          <strong style="font-size: 18px;margin:0;">備註：${
            app.備註 ? app.備註 : "無"
          }</strong>`);
      });
    },
  },
  mounted() {
    const api =
      "https://raw.githubusercontent.com/iam-CJ/SideProject_2/main/e9f075c5-28de-40f2-bf4e-b5cce4757136.json";
    this.$http.get(api).then((response) => {
      this.data = response.data;
      //console.log(this.data);
    });

    openStreetMap = Leaflet.map("map", {
      center: [24.80395, 120.9647],
      zoom: 13,
    });

    Leaflet.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 25,
    }).addTo(openStreetMap);
  },
  watch: {
    speedMeasuringApps() {
      this.updateMap();
    },
  },
};
</script>
<style>
.menus-area {
  width: 100%;
  position: fixed;
  z-index: 999;
  top: 0;
  left: 0;
}
h3 {
  text-align: center;
  margin: 5px 0;
}
.menus {
  width: 100%;
  display: flex;
  background-color: rgba(255, 255, 255, 0.3);
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}
.menu-group {
  margin: 10px;
}
.menu-group > p {
  margin: 0;
}
.menu-group > p > span {
  color: blue;
  font-weight: bolder;
}
.form-select {
  display: unset;
}
#map {
  height: 90vh;
}
</style>
