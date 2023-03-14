<template>
  <div class="home">
    高德地图多个途经点规划
    <div id="container" class="map"></div>
  </div>
</template>

<script>
import gdMap from '@/utils/map';
export default {
  name: 'HomeView',
  data() {
    return {
      map: undefined,
      AMap: undefined,
      // 途经点数据
      mapData: {
        lineData: [],
        waypoints: [], // 途径路线
        path: [
          { lng: '104.1033427953148', lat: '30.65944314998706' },
          { lng: '104.09779062265397', lat: '30.66349924949731' },
          { lng: '104.06879326099397', lat: '30.68088455919872' },
          { lng: '104.04862304920198', lat: '30.701255716384242' },
          { lng: '104.03815170520784', lat: '30.684944371895217' },
          { lng: '104.04167076343538', lat: '30.677193672271905' },
          { lng: '104.02210136646272', lat: '30.66752287966578' },
          { lng: '104.00175949329378', lat: '30.651870385822164' },
          { lng: '104.00622268909456', lat: '30.631193396566303' },
          { lng: '104.03248687976838', lat: '30.609921025588374' },
          { lng: '104.04021164173128', lat: '30.592264363549486' },
          { lng: '104.05574699634553', lat: '30.58672290670932' },
          { lng: '104.06338592761995', lat: '30.568840312466573' },
          { lng: '104.07686134571077', lat: '30.55782831471453' },
          { lng: '104.08226867908479', lat: '30.54452356343893' },
          { lng: '104.07059570545198', lat: '30.55014134751965' },
          { lng: '104.08364197010042', lat: '30.530181956598803' },
          { lng: '104.0858735680008', lat: '30.508369785355946' },
          { lng: '104.07428642505647', lat: '30.494170759836052' },
          { lng: '104.1063871025467', lat: '30.542305927611356' },
          { lng: '104.15119072193147', lat: '30.622921363333386' },
          { lng: '104.14071937793733', lat: '30.629347021877322' },
          { lng: '104.13299461597444', lat: '30.65150119263842' },
          { lng: '104.265', lat: '30.65150119263842' },
          { lng: '104.254', lat: '30.65150119263842' },
          { lng: '104.261', lat: '30.445' },
          { lng: '104.161', lat: '30.65150119263842' },
          { lng: '104.218', lat: '30.65150119263842' },
          { lng: '104.33', lat: '30.787' },
          { lng: '104.66', lat: '30.9686' },
          { lng: '104.44', lat: '30.65150119263842' },
          { lng: '104.041', lat: '30.4541' },
          { lng: '104.11', lat: '30.3975' },
          { lng: '104.47', lat: '30.7844' },
          { lng: '104.63', lat: '30.688411' },
          { lng: '104.85', lat: '30.21212' },
          { lng: '104.789', lat: '30.877' },
          { lng: '104.486', lat: '30.447' },
          { lng: '104.358', lat: '30.95577' },
        ],
      },
      routerValList: [],
      formData: {
        startObj: {
          lng: '104.1228826879883',
          lat: '30.666927651357103',
        },
        endObj: {
          lng: '104.06112750762941',
          lat: '30.729291314872896',
        },
      },
    };
  },
  created() {
    this.initMap();
  },
  mounted() {
    this.getRouterVal(this.mapData.path);
    console.log(this.mapData.path.length, 123123);
  },
  methods: {
    getRouterVal(val) {
      this.routerValList = val;
      this.editRouter = Array.from(val);
      // 有途径网点后重新渲染路线
      this.$nextTick(() => {
        this.driverLine(true);
      });
    },
    //地图节点生成
    driverLine(isFull) {
      // isFull 是否自动居中显示
      this.map = new AMap.Map('container', {
        // resizeEnable: true,
        //地图中心点
        zoom: 12, //地图显示的缩放级别
      });
      let driverlocations = '';
      const that = this;
      const startPoi = new AMap.LngLat(this.formData.startObj.lng, this.formData.startObj.lat);
      const endPoi = new AMap.LngLat(this.formData.endObj.lng, this.formData.endObj.lat);

      const waypointsPoi = this.routerValList.map((item) => {
        let res;
        if (item) {
          res = new AMap.LngLat(item.lng, item.lat);
        } else {
          res = false;
        }
        return res;
      });
      // 当超过十六个途径网点的时候 做拆分获取导航数据
      // 将起点 终点加入到途径网点 用来获取路径
      var drivingRouter = new AMap.Driving({
        // map: this.map,
        // 驾车路线规划策略
        // autoFitView: true
        // policy: AMap.DrivingPolicy.LEAST_DISTANCE
      });
      waypointsPoi.unshift(startPoi);
      waypointsPoi.push(endPoi);
      const length = waypointsPoi.length;
      const midNum = 16;
      const count = Math.ceil(length / midNum); // 向上取整 获取请求次数
      // const lineList = []
      let mid, start, end;
      const tmpplandatas = [];
      for (let i = 0; i < count; i++) {
        start = midNum * i; // 起点
        if (length - start > midNum) {
          end = start + midNum;
          mid = Array.from(waypointsPoi).splice(start + 1, midNum);
        } else {
          end = length - 1;
          mid = Array.from(waypointsPoi).splice(start, end);
        }
        tmpplandatas.push(
          new Promise((resolve, reject) => {
            drivingRouter.search(waypointsPoi[start], waypointsPoi[end], { waypoints: mid }, function (status, result) {
              // result 即是对应的驾车导航信息，相关数据结构文档请参考  https://lbs.amap.com/api/javascript-api/reference/route-search#m_DrivingResult
              if (status === 'complete') {
                // 实际里程赋值
                resolve(result);
                // 具体线路赋值
              } else {
                reject(result);
              }
            });
          })
        );
      }
      // 所有请求结束后执行
      Promise.all(tmpplandatas)
        .then((res) => {
          // 实际里程
          let totalDis;
          const lineData = [];
          res.forEach((item) => {
            totalDis += item.routes[0].distance;
            // 实际路线
            item.routes[0].steps.forEach((item2) => {
              item2.path.forEach((item3) => {
                const list = [];
                list[0] = item3.lng;
                list[1] = item3.lat;
                lineData.push(list);
              });
            });
          });
          if (this.routerValList) {
            const tjList = this.routerValList.map((item) => {
              item.lng = item.lng || item.location.lng;
              item.lat = item.lat || item.location.lat;
              return item;
            });
            this.mapData.waypoints = tjList;
          }
          this.formData.liCheng = (totalDis / 1000).toFixed(2);
          this.mapData.lineData = lineData;
          this.drawLineFn(this.mapData.lineData, isFull);
        })
        .catch((err) => {
          console.log('线路生成失败');
        });

      if (this.routerValList) {
        const tjList = this.routerValList.map((item) => {
          item.lng = item.lng || item.location.lng;
          item.lat = item.lat || item.location.lat;
          return item;
        });
        this.mapData.waypoints = tjList;
      }
    },
    darwMidLineFn(isFull) {
      // 绘制途径点，起、终点
      const startIcon = new this.AMap.Icon({
        // 图标尺寸
        size: new this.AMap.Size(25, 34),
        // 图标的取图地址
        image: '//a.amap.com/jsapi_demos/static/demo-center/icons/dir-marker.png',
        // 图标所用图片大小
        imageSize: new this.AMap.Size(135, 40),
        // 图标取图偏移量
        imageOffset: new this.AMap.Pixel(-9, -3),
      });
      const marker = new this.AMap.Marker({
        icon: startIcon,
        size: [3, 3],
        position: [this.formData.startObj.lng, this.formData.startObj.lat],
        draggable: false,
      });
      // marker.on('dragend', (res) => {
      //   const { lng, lat } = res.lnglat;
      //   this.formData.startObj.lng = lng;
      //   this.formData.startObj.lat = lat;
      //   this.driverLine2(true);
      // });
      const endIcon = new this.AMap.Icon({
        size: new this.AMap.Size(25, 34),
        image: '//a.amap.com/jsapi_demos/static/demo-center/icons/dir-marker.png',
        imageSize: new this.AMap.Size(135, 40),
        imageOffset: new this.AMap.Pixel(-95, -3),
      });
      const marker2 = new this.AMap.Marker({
        icon: endIcon,
        size: [3, 3],
        position: [this.formData.endObj.lng, this.formData.endObj.lat],
        draggable: false,
      });
      // marker2.on('dragend', (res) => {
      //   const { lng, lat } = res.lnglat;
      //   this.formData.endObj.lng = lng;
      //   this.formData.endObj.lat = lat;
      //   this.driverLine2(true);
      // });
      this.map.add([marker, marker2]);
      this.routerValList.forEach((item, index) => {
        const startIcon = new this.AMap.Icon({
          // 图标尺寸
          size: new this.AMap.Size(25, 34),
          // 图标的取图地址
          image: '//a.amap.com/jsapi_demos/static/demo-center/icons/dir-marker.png',
          // 图标所用图片大小
          imageSize: new this.AMap.Size(135, 40),
          // 图标取图偏移量
          imageOffset: new this.AMap.Pixel(-53, -3),
        });
        const marker = new this.AMap.Marker({
          icon: startIcon,
          size: [3, 3],
          position: [item.lng, item.lat],
          draggable: false,
        });
        let routerValIndex;
        // marker.on('mousedown', (res) => {
        //   const list = [this.currentLngLat.lng, this.currentLngLat.lat];
        //   const distanceList = this.routerValList.map((item2) => {
        //     // 获取距离list
        //     if (!item2.lng || !item2.lat) {
        //       this.$message.warning('请检途径点输入是否有效！');
        //     }
        //     const p = [item2.lng, item2.lat];
        //     return this.AMap.GeometryUtil.distance(list, p);
        //   });
        //   const min = Math.min.apply(null, distanceList); // 最小值
        //   routerValIndex = distanceList.indexOf(min);
        //   // console.log(routerValIndex)
        // });
        // marker.on('dragend', (res) => {
        //   const { lng, lat, Q, R } = res.lnglat;
        //   this.routerValList[routerValIndex].lat = lat;
        //   this.routerValList[routerValIndex].lng = lng;
        //   this.routerValList[routerValIndex].location.lat = lat;
        //   this.routerValList[routerValIndex].location.lng = lng;
        //   this.routerValList[routerValIndex].location.Q = Q;
        //   this.routerValList[routerValIndex].location.R = R;
        //   this.driverLine2(true);
        // });
        this.map.add(marker);
      });
      if (isFull) {
        return;
      }
      this.map.setFitView();
    },
    //绘制线路
    drawLineFn(pathData, isFull) {
      const path = pathData.map((item) => {
        return new this.AMap.LngLat(item[0], item[1]);
      });
      var newLine = new this.AMap.Polyline({
        path: path,
        strokeWeight: 8,
        strokeOpacity: 0.8,
        strokeColor: '#0091ea',
        showDir: true,
        lineJoin: 'round',
      });
      this.map.add(newLine);
      this.darwMidLineFn(isFull);
    },
    //初始化地图
    initMap() {
      const that = this;
      gdMap().then(
        (AMap) => {
          that.AMap = AMap;
          that.map = new AMap.Map('container', {
            resizeEnable: true,
            zoom: 13,
          });
          AMap.plugin(['AMap.ToolBar', 'AMap.Scale'], function () {
            that.map.addControl(new AMap.ToolBar());
            that.map.addControl(new AMap.Scale());
          });
          // this.addMapMarker();
        },
        (e) => {
          console.log('地图加载失败', e);
        }
      );
    },
  },
};
</script>
<style lang="scss" scoped>
.map {
  margin: 0 auto;
  width: 1200px;
  height: 700px;
}
</style>
