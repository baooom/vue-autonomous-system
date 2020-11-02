<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="12">
                <!--el-card shadow="hover">
                    地图监控<v-parser></v-parser>
                </el-card-->
                <el-card shadow="hover" >
                    百度地图
                    <div id="container" style="height:600px"></div>
                </el-card>
            </el-col>
            
            <el-col :span="12">
                <el-card shadow="hover">
                    车辆监控<v-car-state @transferVin="getVin"></v-car-state>
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>


<script>
import Schart from 'vue-schart';
import bus from '../common/bus';
//导入threee.js
import vParser from '@/components/common/XodrParser.vue'
import vCarState from '@/components/common/CarState.vue'
import VueResource from 'vue-resource'
import Vue from 'vue'
import GLOBALDATA from '../common/globalData';
Vue.use(VueResource);

export default {
    name: 'dashboard',
    data() {
        return {
            name: localStorage.getItem('ms_username'),
            url:GLOBALDATA.url,
            map: null,
            index:0,
            path:[{
                'lng': 120.11853426,
                'lat': 30.261610015
            }, {
                'lng': 120.118602567,
                'lat': 30.2618395767
            }, {
                'lng': 120.117588913,
                'lat': 30.2621267167
            }, {
                'lng': 120.117404863,
                'lat': 30.26219829
            }, {
                'lng': 120.117244887,
                'lat': 30.2617942267
            }, {
                'lng': 120.116989627,
                'lat': 30.2612456667
            }, {
                'lng': 120.11721955,
                'lat': 30.2613977667
            }],
            line:[],
            selected_car:0
        };
    },
    components: {
        Schart,
        vParser,
        vCarState
    },
    computed: {
        role() {
            return this.name === 'admin' ? '超级管理员' : '普通用户';
        }
    },
    created() {
        this.$nextTick(() => {
            this.init();
            setInterval(this.fetch, 1000);
        })
    },
    methods: {
        init(){
            this.map = new BMap.Map('container');
            var point = new BMap.Point(this.path[0]['lng'], this.path[0]['lat']); 
            this.map.centerAndZoom(point, 19);
            this.map.enableScrollWheelZoom(true); 
            var marker = new BMap.Marker(point);
            this.map.addOverlay(marker);            
            console.log("map init");
        },
        getVin(msg){
            console.log("---------------msg transfer")
            this.selected_car = msg;
        },
        fetch(){
            this.$http.get('http://'+this.url+'/api/v2/info/car/gps/'+this.selected_car).then(response=>{
                if(response.body['code'] == 0){
                    var tmp = response.body.attach;
                    console.log(tmp);
                    var lng = tmp['longitude'];
                    var lat = tmp['latitude'];
                    if(lng < 0.1)
                        return;
                    if(lat < 0.1)
                        return;
                    console.log(lng,lat)
                    var p = new BMap.Point(lng, lat);
                    setTimeout(function(map,array){
                        var convertor = new BMap.Convertor();
                        var pointArr = [];
                        pointArr.push(p);
                        convertor.translate(pointArr, 1, 5, function(data){
                            if(data.status === 0) {
                                map.clearOverlays();
                                array.push(data.points[0]);
                                //var polyline = new BMap.Polyline(array, {strokeColor: 'blue',strokeWeight: 2,strokeOpacity: 0.5});
                                //map.addOverlay(polyline);
                                var marker = new BMap.Marker(data.points[0]);
                                map.addOverlay(marker);
                                var label = new BMap.Label("车辆位置",{offset:new BMap.Size(20,-10)});
                                marker.setLabel(label); //添加百度label
                                map.setCenter(data.points[0]);
                            }
                        });
                    }(this.map, this.line), 1000);
                    this.index = this.index + 1;
                }else{
                    console.log('data not exists')
                }
            },response=>{
                console.log('fail');
            });
        }
    }                            
};

</script>


<style scoped>
.el-row {
    margin-bottom: 20px;
}

.grid-content {
    display: flex;
    align-items: center;
    height: 100px;
}

.grid-cont-right {
    flex: 1;
    text-align: center;
    font-size: 14px;
    color: #999;
}

.grid-num {
    font-size: 30px;
    font-weight: bold;
}

.grid-con-icon {
    font-size: 50px;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    color: #fff;
}

.grid-con-1 .grid-con-icon {
    background: rgb(45, 140, 240);
}

.grid-con-1 .grid-num {
    color: rgb(45, 140, 240);
}

.grid-con-2 .grid-con-icon {
    background: rgb(100, 213, 114);
}

.grid-con-2 .grid-num {
    color: rgb(45, 140, 240);
}

.grid-con-3 .grid-con-icon {
    background: rgb(242, 94, 67);
}

.grid-con-3 .grid-num {
    color: rgb(242, 94, 67);
}

.user-info {
    display: flex;
    align-items: center;
    padding-bottom: 20px;
    border-bottom: 2px solid #ccc;
    margin-bottom: 20px;
}

.user-avator {
    width: 120px;
    height: 120px;
    border-radius: 50%;
}

.user-info-cont {
    padding-left: 50px;
    flex: 1;
    font-size: 14px;
    color: #999;
}

.user-info-cont div:first-child {
    font-size: 30px;
    color: #222;
}

.user-info-list {
    font-size: 14px;
    color: #999;
    line-height: 25px;
}

.user-info-list span {
    margin-left: 70px;
}

.mgb20 {
    margin-bottom: 20px;
}

.todo-item {
    font-size: 14px;
}

.todo-item-del {
    text-decoration: line-through;
    color: #999;
}

.schart {
    width: 100%;
    height: 300px;
}
</style>
