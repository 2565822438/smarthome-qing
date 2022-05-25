<template id="qing">
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
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">温度</div>
              <div class="data-value">{{Temp}}℃</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">湿度</div>
              <div class="data-value">{{Hum}}%</div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">MQ135</div>
              <div class="data-value">{{MQ135}}</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">灯光</div>
              <div class="data-value">
                <switch @change="onledChange" :checked="led" color="#3d7ef9" />
              </div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">插座</div>
              <div class="data-value">
                <switch @change="onrelayChange" :checked="relay" color="#3d7ef9" />
              </div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">门</div>
              <div class="data-value">
                <switch @change="ondoorChange" :checked="door" color="#3d7ef9" />
              </div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png">
            <div class="data-text">
              <div class="data-title">安防</div>
              <div class="data-value">
                <switch @change="onsecurityChange" :checked="security" color="#3d7ef9" />
              </div>
            </div>
          </div>
          
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { connect } from 'mqtt/dist/mqtt.js'
const mqttUrl = 'wxs://mqtt.yyqing.net:8084/mqtt'
export default {
  data () {
    return {
      client:{},
      Temp:0,
      Hum:0,
      MQ135:0,
      led:false,
      relay:false,
      security:false,
      area:'请求中',//城区
      city:'请求中',//城市
      airText:'请求中',//空气优良
      airValue:0,//空气质量指数
      weather:'请求中',
      weatherAdvice:'请求中',
    }
  },
  components: {

  },

  methods: {
    onledChange(event) {
      var that = this
      console.log(event.mp.detail)
      let sw = event.mp.detail.value
      that.led = sw
      if (sw) {
        that.client.publish('/smarthome/qing','{"target":"LED","value":1}')
      } else {
        that.client.publish('/smarthome/qing','{"target":"LED","value":0}')
      }
    },
    onrelayChange(event) {
      var that = this
      console.log(event.mp.detail)
      let sw = event.mp.detail.value
      that.relay = sw
      if (sw) {
        that.client.publish('/smarthome/qing','{"target":"RELAY","value":1}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,插座开')
          }
        })
      } else {
        that.client.publish('/smarthome/qing','{"target":"RELAY","value":0}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,插座关')
          }
        })
      }
    },
    ondoorChange(event) {
      var that = this
      console.log(event.mp.detail)
      let sw = event.mp.detail.value
      that.door = sw
      if (sw) {
        that.client.publish('/smarthome/qing','{"target":"DOOR","value":1}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,门开')
          }
        })
      } else {
        that.client.publish('/smarthome/qing','{"target":"DOOR","value":0}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,门关')
          }
        })
      }
    },
    onsecurityChange(event) {
      var that = this
      console.log(event.mp.detail)
      let sw = event.mp.detail.value
      that.security = sw
      if (sw) {
        that.client.publish('/smarthome/qing','{"target":"security","value":1}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,安防模式开')
          }
        })
      } else {
        that.client.publish('/smarthome/qing','{"target":"security","value":0}',function(err){
          if(err){
            console.log(err)
          }
          else {
            console.log('发送成功,安防模式关')
          }
        })
      }
    },
  },

  // created () {
  //   // let app = getApp()
  // }
  onShow () {
    var that = this
    that.client = connect(mqttUrl)
    that.client.on('connect', function () {
      console.log('连接成功')
      that.client.subscribe('/smarthome/qing', function (err) {
        if (!err) {
          console.log('订阅成功设备上行数据')
        }
      })
    })
    that.client.on('message', function (topic, message) {
      console.log(topic)
      console.log(message)
      let dataFromDev = {}
      dataFromDev = JSON.parse(message)
      console.log(dataFromDev)
      that.Temp = dataFromDev.Temp
      that.Hum = dataFromDev.Hum
      that.MQ135 = dataFromDev.MQ135
      that.led = dataFromDev.LED
      that.relay = dataFromDev.RELAY
      that.door = dataFromDev.DOOR
    });
    wx.getLocation({
      type: 'wgs84',
      success (res) {
        const latitude =Math.floor(res.latitude*100)/100
        const longitude =Math.floor(res.longitude*100)/100
        const key = "e5441a0fe3b7427d8b4bbb0012610885";
        wx.request({
          url: `https://free-api.qweather.com/s6/weather/now?location=${longitude},${latitude}&key=${key}`, //天气数据api接口
          success (res) {
            console.log(res.data)
            const { basic ,now} = res.data.HeWeather6[0];
            console.log(basic)
            console.log(now)
            that.area = basic.location
            that.city = basic.parent_city
            that.weather = now.cond_txt
          }
        })
        wx.request({
          url: `https://devapi.qweather.com/v7/air/now?location=${longitude},${latitude}&key=${key}`, //空气情况api接口
          success (res) {
            console.log(res.data)
            that.airText = res.data.now.category
            that.airValue = res.data.now.aqi
          }
        })
        wx.request({
          url: `https://devapi.qweather.com/v7/indices/1d?type=1,3,8,13,15&location=${longitude},${latitude}&key=${key}`, //空气情况api接口
          success (res) {
            console.log(res.data)
            that.weatherAdvice = res.data.daily[4].text
          }
        })
      }
    })
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
padding: 15px;
.header-wrapper{
  background-color: #3d7ef9;
  border-radius: 20px;
  color: #fff;
  box-shadow: #d6d6d6 0px 0px 5px;
  padding:15px 30px;
  .header-title{
    display: flex;
    justify-content: space-between;
  }
  .header-text{
    font-size: 32px;
    font-weight: 400;
    display: flex;
    justify-content: space-between;
  }
  .header-advice{
    margin-top: 20px;
    font-size: 12px;
  }
}
.data-wrapper{
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  .data{
    background-color: #fff;
    width: 150px;
    height: 80px;
    border-radius: 20px;
    display: flex;
    justify-content: space-around;
    padding: 0 8px;
    box-shadow: #d6d6d6 0px 0px 5px;
    .data-logo{
      height: 36px;
      width: 36px;
      margin-top: 15px;
    }
    .data-text{
      margin-top: 15px;
      color: #7f7f7f;
      .data-value{
        font-size: 26px;
      }
    }
  }
}
}
</style>
