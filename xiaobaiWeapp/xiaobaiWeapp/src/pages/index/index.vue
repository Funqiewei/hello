<!--
 * @Author: your name
 * @Date: 2020-05-27 20:16:48
 * @LastEditTime: 2020-05-29 19:20:14
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \xiaobaiWeapp\src\pages\index\index.vue
--> 
<template>
  <div class="wrapper">
    <div class="header-wrapper">
      <div class="header-title">
        <span>空气质量-{{airText}}</span>
        <span>{{area}}-{{city}}</span>
      </div>
      <div class="header-text">
        <span>{{airValue}}</span>
        <span>{{weather}}</span>
      </div>
      <div class="weather-advice">{{weatherAdvice}}</div>
    </div>
    <div class="body-wrapper">
      <div class="body">
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png" />
            <div class="data-text">
              <div class="data-title">温度</div>
              <div class="data-value">{{Temp}}℃</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/shidu.png" />
            <div class="data-text">
              <div class="data-title">湿度</div>
              <div class="data-value">{{Hum}}%</div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/guangzhaodu.png" />
            <div class="data-text">
              <div class="data-title">光照度</div>
              <div class="data-value">{{Light}}Lx</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/led.png" />
            <div class="data-text">
              <div class="data-title">客厅灯</div>
              <div class="data-value">
                <switch @change="onLedChange" :checked="Led" color="#3d7ef9" />
              </div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/beep.png" />
            <div class="data-text">
              <div class="data-title">报警器</div>
              <div class="data-value">
                <switch @change="onBeepChange" :checked="Beep" color="#3d7ef9" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { connect } from "mqtt";

const mqttUrl = "wxs://mqtt.mqttssledu.xyz:8084/mqtt";

export default {
  data() {
    return {
      client: {},
      Temp: 0,
      Hum: 0,
      Light: 0,
      Led: false,
      Beep: false,
      area: "请求中", //城区
      city: "请求中", //城市
      airText: "请求中", //空气优良
      airValue: 0, //空气指数
      weather: "请求中", //天气
      weatherAdvice: "请求中" //天气建议
    };
  },

  components: {},

  methods: {
    onLedChange(event) {
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Led = sw;
      if (sw) {
        that.client.publish(
          "/mysmarthome/sub",
          '{"target":"LED","value":1}',
          function(err) {
            if (!err) {
              console.log("成功下发命令——开灯");
            }
          }
        );
      } else {
        that.client.publish(
          "/mysmarthome/sub",
          '{"target":"LED","value":0}',
          function(err) {
            if (!err) {
              console.log("成功下发命令——关灯");
            }
          }
        );
      }
    },
    onBeepChange(event) {
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Beep = sw;
      if (sw) {
        that.client.publish(
          "/mysmarthome/sub",
          '{"target":"BEEP","value":1}',
          function(err) {
            if (!err) {
              console.log("成功下发命令——打开报警器");
            }
          }
        );
      } else {
        that.client.publish(
          "/mysmarthome/sub",
          '{"target":"BEEP","value":0}',
          function(err) {
            if (!err) {
              console.log("成功下发命令——关闭报警器");
            }
          }
        );
      }
    }
  },
  // created() {
  //   // let app = getApp()
  // },
  onShow() {
    var that = this;
    that.client = connect(mqttUrl);
    that.client.on("connect", function() {
      console.log("成功连接mqtt服务器！");
      that.client.subscribe("/mysmarthome/pub", function(err) {
        if (!err) {
          console.log("成功订阅设备上行数据Topic!");
        }
      });
    });
    that.client.on("message", function(topic, message) {
      console.log(topic);
      // message是16进制的Buffer字节流
      let dataFromDev = {};
      dataFromDev = JSON.parse(message);
      console.log(dataFromDev);
      that.Temp = dataFromDev.Temp;
      that.Hum = dataFromDev.Hum;
      that.Light = dataFromDev.Light;
      that.Led = dataFromDev.Led;
      that.Beep = dataFromDev.Beep;
    });
    wx.getLocation({
      type: "wgs84",
      success(res) {
        const latitude = res.latitude;
        const longitude = res.longitude;
        const key = "8d90114ce9334ccbbcf6dec72f534bd7";
        wx.request({
          url: `https://free-api.heweather.net/s6/weather/now?location=${longitude},${latitude}&key=${key}`, //获取天气数据的api接口地址
          success(res) {
            // console.log(res.data);
            const { basic, now } = res.data.HeWeather6[0];
            // console.log(basic);
            // console.log(now);
            that.area = basic.location;
            that.city = basic.parent_city;
            that.weather = now.cond_txt;
            wx.request({
              url: `https://free-api.heweather.net/s6/air/now?location=${that.city}&key=${key}`, //获取空气数据的api接口地址
              success(res) {
                // console.log(res.data);
                const { air_now_city } = res.data.HeWeather6[0];
                // console.log(air_now_city);
                const { aqi, qlty } = air_now_city;
                that.airText = qlty;
                that.airValue = aqi;
              }
            });
          }
        });
        wx.request({
          url: `https://free-api.heweather.net/s6/weather/lifestyle?location=${longitude},${latitude}&key=${key}`, //获取空气数据的api接口地址
          success(res) {
            console.log(res.data);
            const { lifestyle} = res.data.HeWeather6[0];
            console.log(lifestyle)
            that.weatherAdvice = lifestyle[1].txt
          }
        });
      }
    });
  }
};
</script>

<style lang="scss" scoped>
.wrapper {
  padding: 15px;
  .header-wrapper {
    background-color: #3d7ef9;
    border-radius: 20px;
    color: #fff;
    box-shadow: #d6d6d6 0px 0px 5px;
    padding: 15px 30px;
    .header-title {
      display: flex;
      justify-content: space-between;
    }
    .header-text {
      font-size: 32px;
      font-weight: 400;
      display: flex;
      justify-content: space-between;
    }
    .weather-advice {
      margin-top: 20px;
      font-size: 12px;
    }
  }
  .data-wrapper {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    .data {
      background-color: #fff;
      width: 150px;
      height: 80px;
      border-radius: 20px;
      display: flex;
      justify-content: space-around;
      padding: 0 8px;
      box-shadow: #d6d6d6 0px 0px 5px;
      .data-logo {
        height: 36px;
        width: 36px;
        margin-top: 15px;
      }
      .data-text {
        margin-top: 15px;
        color: #7f7f7f;
        .data-title {
          text-align: right;
        }
        .data-value {
          font-size: 26px;
        }
      }
    }
  }
}
</style>
