<template>
  <div class="home">
    <h1>郵便番号から住所と天気を検索</h1>
    <input type="text" v-model="zip" placeholder="半角7桁">
    <button @click="getAddWeather()">住所自動入力</button>
    <p>Address:{{ address }}</p>
      <ul v-if="show">
        <li>天気:{{main}}</li>
        <li>温度:{{temp}}°</li>
        <li>湿度:{{humidity}}%</li>
        <li>風速:{{speed}}m</li>
        <li>緯度:{{latitude}}</li>
        <li>経度:{{longitude}}</li>
      </ul>         
  </div>
</template>

<script>
// apiのデータが取れたurl→これを分解してみる。
// https://apis.postcode-jp.com/api/v4/postcodes/1300021?apiKey=xfkCrjkDqIx47ni6rO1zPntSSr59KruMf5GilPL

import axios from "axios";
const ZIP_API_URL = "https://apis.postcode-jp.com/api/v4/postcodes/";
const ZIP_API_KEY = "?" +"apiKey=xfkCrjkDqIx47ni6rO1zPntSSr59KruMf5GilPL";
const WEATHER_API_URL ="https://api.openweathermap.org/data/2.5/weather?zip=";
const WEATHER_API_KEY= ",jp&units=metric&"+ "appid=e777b062735aa51d7ea15cb6e0ca541a";

export default {
  data() {
    return {
      zip:"",
      address:"",
      hyphenedZip:"",
      name:"",
      main:"",
      temp:"",
      humidity:"",
      speed:"",
      latitude:"",
      longitude:"",
      show: false,
    };
  },
  watch: {
    zip(zip){
      this.zip = this.digitCheck(zip);
    }
  },

  methods: {
     digitCheck(zip){
      return zip.length > 7? zip.slice(0,-1) : zip;
    },
    async getAddress() {
      let item = await axios.get(ZIP_API_URL + Number(this.zip) + ZIP_API_KEY);
      console.log(item.data);
      this.data = item.data;
      this.address = this.data[0].allAddress;
      // console.log(this.data[0].postcode);
      let firstThree = this.data[0].postcode.slice(0,3);
      // console.log(firstThree);
      let lastFour = this.data[0].postcode.slice(3);
      // console.log(lastFour);
      this.hyphenedZip = firstThree + "-" + lastFour;
      // console.log(hyphenedZip);
    },
    async getWeather(){
      await this.getAddress();
      console.log(this.hyphenedZip);
      let param  = await axios.get(WEATHER_API_URL + this.hyphenedZip + WEATHER_API_KEY);
      this.data = param.data;
      this.name = this.data.name;
      this.main = this.data.weather[0].main;
      this.temp = this.data.main.temp;
      this.humidity = this.data.main.humidity;
      this.speed = this.data.wind.speed;
      this.latitude = this.data.coord.lat
      this.longitude = this.data.coord.lon
      console.log(param.data)
    },
    userclick (){
      this.show = !this.show;
    },
    getAddWeather(){
      this.getAddress();
      this.getWeather();
      this.userclick();
    }
  }
}

</script>

<style scoped>
.home {
  margin: 20px;
  line-height: 2;
}

li {
  list-style: none;
}

</style>