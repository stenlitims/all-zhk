<template>
  <div id="app">

    <div class="row">
      <div class="col-md-6 col-lg-4">
        <div class="form-group">
          <select class="form-control" @change="chzk()" v-model="zhk">
            <option  v-for="(option, i)  in listZ"  :key="i" v-bind:value="option">{{option.name}}</option>
          </select>
        </div>
      </div>
    </div>
  

    <div class="sections">
      <div class="item" v-for="(item, i) in sections" :key="i">
        <div class="sec-title">{{lang.section}} {{i}}</div>
        <div class="flats">
          <div class="level" v-for="(item2, index) in item" :key="index">
            <div class="num">{{item2.lev}}</div>

              <div class="flat" v-for="(item3, index) in item2.flats" :key="index" :class="{ active: ap(i, item2.lev, index).rooms != 0 }" @mouseover="openp(ap(i, item2.lev, index), $event)" @mouseleave="hidep()">
                    {{ap(i, item2.lev, index).rooms}}
              </div>

          </div>
        </div>
      </div>
    </div>
   

    <div :style="pos" :class="popclass" class="toltip">
      <div class="img">
        <img :src="pop.img2" alt="">
      </div>
      <div class="room">
        {{pop.rooms}} - {{lang.rooms}}
      </div>
      <div class="sq">
        {{pop.square}} м<sup>2</sup>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Vue from "vue";
import VueLodash from "vue-lodash";

const options = { name: "lodash" };
Vue.use(VueLodash, options);

let lang = {
  ru: {
    section: "Секция",
    rooms: "комнатная"
  },
  uk: {
    section: "Секція",
    rooms: "кімнатна"
  }
};

let curLang = $("body").data("lang");

let prlist = {};

$(".js-pr-list > div").each(function() {
  let tempData = $(this).data();
  if (tempData.api) {
    prlist[tempData.id] = tempData;
  }
});
let keys = Object.keys(prlist);

//console.log(prlist);

export default {
  name: "App",
  data() {
    return {
      flats: [],
      pop: {},
      prlist: prlist,
      popclass: "",
      zhk: prlist[keys[0]],
      lang: lang[curLang],
      pos: {
        left: "10px",
        top: "10px"
      },
      allSec: allSec,
      sections: allSec[keys[0]]
    };
  },
  created() {
    this.loadFlats(this.zhk.api);
  },
  computed: {
    listZ() {
      return this.lodash.sortBy(prlist, [
        function(o) {
          return o.name;
        }
      ]);
    }
  },
  methods: {
    loadFlats(api) {
      axios
        .get(api)
        .then(response => {
          let flats = {};
          for (let item in this.sections) {
            this.sections[item].forEach(el => {
              let data = Vue.lodash.filter(response.data, {
                section: +item,
                floor: +el.lev
              });
              data.forEach((item2, i) => {
                flats[item + "_" + el.lev + "_" + i] = item2;
              });
            });
          }
          this.flats = flats;
        })
        .catch(e => {
          this.erros.push(e);
        });
    },
    sort(s) {
      if (s === this.currentSort) {
        this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc";
      }
      this.currentSort = s;
    },
    flat(n) {
      let i = 1,
        d = [];
      for (; i <= n; i++) {
        d.push(i);
      }
      return d.reverse();
    },
    ap(sec, lev, n) {
      let name = sec + "_" + lev + "_" + n;
      let data = this.flats[name];
      if (!data) {
        data = {
          rooms: " "
        };
      }
      return data;
    },
    chzk() {
      this.loadFlats(this.zhk.api);
      this.sections = allSec[this.zhk.id];
    },
    openp(ob, e) {
      if (!ob.section) return;
      // console.log(e);
      this.pop = ob;
      this.popclass = "active";
      this.pos.left = e.target.offsetLeft + "px";
      this.pos.top = e.target.offsetTop - window.pageYOffset + "px";
    },
    hidep() {
      this.popclass = "";
    }
  }
};
</script>

<style lang="scss">
.sections {
  margin-top: 20px;
  display: flex;
  flex-wrap: wrap;
  margin-left: -20px;
  margin-right: -20px;
  .item {
    max-width: 33.33%;
    flex-basis: 0;
    flex-grow: 1;
    padding-left: 20px;
    padding-right: 20px;
    margin-bottom: 35px;
    flex-direction: column;
    justify-content: flex-end;
    display: flex;
  }
  .level {
    display: flex;
    margin-right: -8px;
    > div {
      width: 26px;
      height: 26px;
      margin-right: 8px;
      margin-bottom: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 12px;
    }
    .flat {
      border-radius: 3px;
      border: 1px solid #e6e6e6;
      opacity: 0.5;
      user-select: none; 
      &.active {
        cursor: pointer;
        opacity: 1;
        transition: all 0.3s ease;
        &:hover {
          color: #fff;
          background: #f49e47;
          border-color: #f49e47;
        }
      }
    }
  }
  .sec-title {
    font-weight: bold;
    font-size: 16px;
    margin-bottom: 15px;
  }
}
.wrap-sections {
  margin-top: 20px;
  h2 {
    margin-bottom: 30px;
    padding-bottom: 0;
  }
}
.toltip {
  position: fixed;
  background: #fff;
  box-shadow: 0 0 3px 0 rgba(0, 0, 0, 0.4);
  padding: 10px;
  border-radius: 3px;
  opacity: 0;
  visibility: hidden;
  transition: all 0.2s ease;
  margin-top: 38px;
  transform: translate(-50%, 0);
  margin-left: 13px;
  width: 140px;
  z-index: 500;
  &:before,
  &:after {
    content: "";
    position: absolute;
    left: 50%;
    top: -7px;
    transform: translate(-50%, 0);
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 0 10px 7px 10px;
    border-color: transparent transparent #fff transparent;
  }
  &:before {
    border-color: transparent transparent rgba(0, 0, 0, 0.3) transparent;
    top: -8px;
  }
  &.active {
    opacity: 1;
    visibility: visible;
  }
  .img {
    text-align: center;
    margin-bottom: 5px;
    img {
      max-height: 100px;
      max-width: 100px;
    }
  }
  .room {
    font-weight: bold;
    margin-bottom: 10px;
  }
}

@media (max-width: 991px) {
  .sections .item {
    max-width: 50%;
    min-width: 50%;
  }
}

@media (max-width: 700px) {
  .sections .item {
    max-width: 100%;
    min-width: 100%;
  }
}
</style>
