<template>
  <div>
    <div class="event-searchbar">
      <input
        type="text"
        placeholder="キーワードを入力"
        size="40"
        v-model="keyword"
        @focus="focus"
        @keyup.enter="searchEvents"
        @keypress="setCanMessageSubmit"
      />
      <button class="event-searchbtn" @click="searchEvents">検索</button>
    </div>
    <div class="event-container">
      <div class="event-sidebar">
        <p>総ヒット件数: {{eventHitCount}}件</p>
        <div v-if="eventDatas.length != 0">
          <div v-for="(eventData,index) in eventDatas" :key="eventData.id">
            <div v-on:click="select($event, index)">
              <p>
                <a :href="eventData['event_url']">{{eventData['title']}}</a>
              </p>
              <ul>
                <li>住所：{{eventData['address']}}</li>
              </ul>
            </div>
            <br />
          </div>
        </div>
        <div v-else>
          <p>{{message}}</p>
        </div>
      </div>
      <div class="event-main">
        <div id="ymap" style="width:100%; height:800px"></div>
      </div>
    </div>
  </div>
</template>

<script>
const axios = require("axios");
import yahooMapLoader from "yahoo-map-loader";

let url = "http://localhost:3000/api/v1/event?keyword=";

export default {
  data() {
    return {
      keyword: "",
      eventHitCount: 0,
      eventDatas: [],
      message: "",
      ymap: null,
      canMessageSubmit: false
    };
  },
  methods: {
    select: function(e, idx) {
      const ed = this.eventDatas[idx];
      yahooMapLoader.load(Y => {
        this.ymap.drawMap(
          new Y.LatLng(ed["lat"], ed["lon"]),
          15,
          Y.LayerSetId.NORMAL
        );
      });
    },
    setCanMessageSubmit() {
      this.canMessageSubmit = true;
    },
    searchEvents() {
      if (!this.canMessageSubmit) {
        return;
      }
      this.eventHitCount = 0;
      this.eventDatas = [];
      this.ymap.clearFeatures();
      this.canMessageSubmit = false;

      axios
        .get(url + this.keyword)
        .then(res => {
          if (res.data.results_returned == 0) {
            this.message = "no data";
            return;
          }
          this.eventHitCount = res.data.results_available;
          this.eventDatas = res.data.events;
          this.plotEventsOnMap();
        })
        .catch(error => {
          this.message = "error";
          console.log(error);
        });
    },
    focus() {
      this.message = "";
    },
    plotEventsOnMap() {
      const ed0 = this.eventDatas[0];
      yahooMapLoader.load(Y => {
        this.ymap.drawMap(
          new Y.LatLng(ed0["lat"], ed0["lon"]),
          15,
          Y.LayerSetId.NORMAL
        );
        this.ymap.clearFeatures();
        var markers = [];
        for (let i = 0; i < this.eventDatas.length; i++) {
          const ed = this.eventDatas[i];
          markers.push(
            new Y.Marker(new Y.LatLng(ed["lat"], ed["lon"]), {
              title: ed["title"]
            })
          );
        }
        this.ymap.addFeatures(markers);
      });
    },
    loadMap() {
      yahooMapLoader.appId = process.env.YMAP_API_KEY;
      yahooMapLoader.load(Y => {
        var map_options = {
          configure: {
            doubleClickZoom: true,
            scrollWheelZoom: true
          }
        };
        this.ymap = new Y.Map("ymap", map_options);
        this.ymap.drawMap(
          new Y.LatLng(35.681236, 139.767125),
          15,
          Y.LayerSetId.NORMAL
        );
        this.ymap.addControl(new Y.LayerSetControl());
        this.ymap.addControl(new Y.ScaleControl());
        this.ymap.addControl(new Y.SliderZoomControlHorizontal());
        this.ymap.clearFeatures();
      });
    }
  },
  mounted: function() {
    this.loadMap();
  }
};
</script>

<style scoped>
.event-searchbar {
  width: 380px;
  margin: 10px;
  padding: 10px;
  background-color: #444;
}

.event-searchbtn {
  background-color: #666;
  padding: 3px;
}

.event-container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
}

.event-sidebar {
  float: left;
  width: 400px;
  padding: 10px;
}

.event-main {
  flex: 1;
  position: relative;
}

.event-title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.event-subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.event-links {
  padding-top: 15px;
}
</style>

<style>
div p.yolp-tlchp {
  color: black;
}
</style>