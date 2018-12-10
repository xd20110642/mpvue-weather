<template>
  <div class="card">
    <div class="top" v-if="show">
      <p>{{shuju.data.city}}</p>
    </div>
    <div class="wether">
      {{shuju.data.wendu}}°C
    </div>
    <div class="warn-mess">
      <p>{{shuju.data.ganmao}}</p>
    </div>
    <div class="yestoday-mess">
       <p>昨天:{{shuju.data.yesterday.date}}</p>
       <p><span>{{shuju.data.yesterday.high}}</span> <span>{{shuju.data.yesterday.low}}</span></p>
    </div>
    <div class="fullter">
        <div class="text-mess" v-for="(item,index) of shuju.data.forecast" :key="index">
          <p>{{item.date}}</p>
          <p>{{item.type}}</p>
          <p>{{item.high}}</p>
          <p>{{item.low}}</p>
          <p>{{item.fengxiang}}</p>
          <!-- <p>{{item.fengli}}</p> -->
        </div>
      
    </div>
    <div class="chaxun">
      <input type="text" placeholder="请输入查询城市" v-model.lazy="cityname">
      <button @click="go">确定</button>
    </div>
  </div>
</template>

<script>
const url='https://restapi.amap.com/v3/geocode/regeo?';
const url1='http://wthrcdn.etouch.cn/weather_mini?='
export default {
  data(){
    return {
        shuju:null,
        lottery:{},//存储定位信息
        citymy:null,
        show:false,
        cityname:''
    }
  },
  methods:{
    // 获取地理定位
    getLocation(){
      wx.getLocation({
        type: 'wgs84', //默认为 wgs84 返回 gps 坐标，gcj02 返回可用于wx.openLocation的坐标,
        success: res => {
          const latitude = res.latitude
          const longitude = res.longitude
          const speed = res.speed
          const accuracy = res.accuracy;
          this.lottery={
            latitude,
            longitude,
            speed,
            accuracy
          };
          console.log(this.lottery);
          this.jiema()
        },
        fail: () => {
          console.log("getLocation failed");
        }
      });
    },
    // 逆解码
    jiema(){
      wx.request({
        url: url, //开发者服务器接口地址",
        data: {
          key:'51ad107f7a94c2858ff9b9a0107c96be',
          location:`${this.lottery.longitude},${this.lottery.latitude}`,
        }, //请求的参数",
        method: 'GET',
        dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
        success: res => {
          console.log(res);
          this.citymy=res.data.regeocode.addressComponent.city.replace('市','');
           this.getMess();
        },
        fail: () => {
          this.city="北京";
           this.getMess();
        },
        complete: () => {
         
        }
      });
    },
    // 搜索天气
    getMess(){
      wx.request({
        url: url1, //开发者服务器接口地址",
        data: {
          city:this.citymy
        }, //请求的参数",
        method: 'GET',
        dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
        success: res => {
          console.log(res)
          this.shuju=res.data;
          this.show=!this.show
          console.log(this.shuju)
        },
        fail: () => {},
        complete: () => {}
      });
    },
    // 手动搜索
    go(){
      wx.request({
        url: url1, //开发者服务器接口地址",
        data: {
          city:this.cityname
        }, //请求的参数",
        method: 'GET',
        dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
        success: res => {
          console.log(res.data);
          this.shuju={};
          this.shuju=res.data;
         
        },
        fail: () => {},
        complete: () => {}
      });
    }
  },
  filters:{
    shanchu(ele){
      if(!ele) return ele;
      let a=ele.replace('市','');
      return a;
    }
  },
  // 页面加载完毕 调用
  onReady(){
    this.getLocation();
  },
  // 使用分享
  
}
</script>

<style scoped>
  .card{
    width: 100%;
    height: 100%;
    overflow: hidden;
    font-family: 微软雅黑,宋体
  }
  .top{
    width: 90%;
    margin: auto;
    border-bottom: 1px solid;
  }
  .wether{
    width: 100%;
    font-size: 20px;
    font-weight: bold;
    text-align: center;
    margin: 20px auto;
  }
  .warn-mess{
    width: 100%;
    margin: auto;
    text-align: center;
  
  }
  .yestoday-mess{
    width: 90%;
    margin: auto;
    margin-top: 10px;
    /* text-align: center; */
    border: 1px solid;
  }
  .yestoday-mess p{
    margin-left: 5px;
  }
  .fullter{
    width: 90%;
    margin: auto;
    margin-top: 10px;
    display: flex;
    justify-content: space-between;
  }
  .fullter .text-mess{
    flex: 1;
    text-align: center;
    border: 1px solid;
    margin: 0 2px;
  }
  .fullter .text-mess p{
      margin: 5px auto;
      font-size: 16px;
  }
  .chaxun{
    width: 90%;
    display: flex;
    margin: auto;
    margin-top: 10px;
  }
  .chaxun input{
    flex: 0 0 70%;
    height: 50px;
    border: 1px solid;
    margin-right: 5px;
  }
  .chaxun button{
    flex: 1;
  }
</style>


