<template>
    <div>
        <el-row :gutter="10">
            <el-col :span="12">
            
            <el-card shadow="hover" style="height:155px;">
                <el-row :gutter="20">
                    车辆选择
                    <el-select placeholder="车辆VIN" v-model="selected_car" @change="selectTrigger(selected_car)">
                        <el-option v-for="(item, index) in connected_car" v-bind:key="index" v-bind:label="item" v-bind:value="item"></el-option>
                    </el-select>
                </el-row>
                <el-row :gutter="20">
                    站点选择
                    <el-select placeholder="站点" v-model="select_station">
                        <el-option v-for="(item, index) in station_info" v-bind:key="index" v-bind:label="item['name']" v-bind:value="item['name']"></el-option>
                    </el-select>
                </el-row>
                <el-row>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item>
                                <el-button type="primary" @click="onSubmit">提交</el-button>
                                <el-button>取消</el-button>
                        </el-form-item>
                    </el-form>
                </el-row>
            </el-card>
            </el-col>

            <el-col :span="12">
            <el-card shadow="hover"  style="height:155px;">
                车辆信息
                <div v-if="info_avail == false">
                    Please select a car!
                </div>
                <div v-else>
                    <div>position-x:{{(selected_car_pos["x"]).toFixed(3)}}</div>
                    <div>position-y:{{(selected_car_pos["y"]).toFixed(3)}}</div>
                    <div>position-z:{{(selected_car_pos["z"]).toFixed(3)}}</div>
                </div>
            </el-card>
            </el-col>
        </el-row>
        
        <el-card shadow="hover" style="height:441px;">
            云远程监控
            <div id="play"></div>
        </el-card>
    </div>
</template>
<script>

import VueResource from 'vue-resource'
import Vue from 'vue'
import GLOBALDATA from '../common/globalData'
Vue.use(VueResource);



export default {
    created(){
        this.$nextTick(() => {
            this.init();
            setInterval(this.fetch, 1000);
        })
    },
    data() {
        return {
            url:GLOBALDATA.url,
            vin:'0',
            info_avail:false,
            connected_car:[],
            selected_car:"",
            selected_car_pos:{},
            virtual_selected_car:"",
            station_info:[],
            select_station:"",
            form: {
                name: '',
                region: '',
                date1: '',
                date2: '',
                delivery: true,
                type: ['步步高'],
                resource: '小天才',
                desc: '',
                options: []
            }
        }
    },
    methods:{
        init(){
            var objectPlayer=new aodianPlayer({
                container:'play',//播放器容器ID，必要参数
                rtmpUrl: 'rtmp://47821.lssplay.aodianyun.com/testsmartcar/stream',//控制台开通的APP rtmp地址，必要参数
                hlsUrl: 'http://47821.hlsplay.aodianyun.com/testsmartcar/stream.m3u8',//控制台开通的APP hls地址，必要参数
                /* 以下为可选参数*/
                width: '720',//播放器宽度，可用数字、百分比等
                height: '480',//播放器高度，可用数字、百分比等
                autostart: true,//是否自动播放，默认为false
                bufferlength: '1',//视频缓冲时间，默认为3秒。hls不支持！手机端不支持
                maxbufferlength: '2',//最大视频缓冲时间，默认为2秒。hls不支持！手机端不支持
                stretching: '1',//设置全屏模式,1代表按比例撑满至全屏,2代表铺满全屏,3代表视频原始大小,默认值为1。hls初始设置不支持，手机端不支持
                controlbardisplay: 'enable',//是否显示控制栏，值为：disable、enable默认为disable。
                //adveDeAddr: '',//封面图片链接
                //adveWidth: 320,//封面图宽度
                //adveHeight: 240,//封面图高度
                //adveReAddr: '',//封面图点击链接
                //isclickplay: false,//是否单击播放，默认为false
                isfullscreen: true//是否双击全屏，默认为true
            });
            console.log(objectPlayer)
        },
        fetch(){
            //获取车辆的信息

            this.$http.get('http://'+this.url+'/api/v2/info/car/' + this.vin  + '/position').then(response=>{
                if(response.body["code"] == 0){
                    console.log("success");
                    this.info_avail = true;
                    this.selected_car_pos = response.body.attach;

                }else{
                    console.log("data not exists");
                    this.info_avail = false;
                }
            } , response=>{
                console.log("fail");
            });
            //获取当前所有连接的车辆
            this.$http.get('http://'+this.url+'/api/v2/info/car/connected').then(response=>{
                if(response.body["code"] == 0){
                    this.connected_car = response.body.attach;
                }else{
                    this.connected_car = [];
                }
            }, response=>{
                console.log("fail");
            });
        },
        onSubmit() {
            //点击提交之后post方法
            console.log("submit");
            var x = 0 , y = 0;
            for(var i=0;i<this.station_info.length;i++){
                if(this.station_info[i]["name"] == this.select_station){
                    x = parseFloat(this.station_info[i]["x"]);
                    y = parseFloat(this.station_info[i]["y"]);
                    break;
                }
            }
            this.$http.post('http://'+this.url+'/api/v2/auto/car/trj/station',{
                vin:this.selected_car,
                goal:this.select_station
            },{emulateJSON: true}).then(response=>{
                if(response.body["code"] == 0){
                    console.log("success");
                }else{
                    console.log("data not exists");
                }
            } , response=>{
                console.log("fail");
            });
        },
        chooseRemote(){
            //选择远程自动驾驶车辆
            this.$http.post('http://'+this.url+'/api/v2/info/car/choose/remote',{
                vin:this.virtual_selected_car
            },{emulateJSON: true}).then(response=>{
                if(response.body["code"] == 0){
                    console.log("success");
                }else{
                    console.log("data not exists");
                }
            } , response=>{
                console.log("fail");
            });
        },
        selectTrigger(val) {
            console.log("trigger")
            //事件触发传递数据
            this.$emit('transferVin', val);
            //设置vin
            this.vin = String(val);
            //获取vin对应的park 然后再查询对应的站点
            this.$http.get('http://' + this.url + '/api/v2/info/car/' + String(val) + '/stations').then(response=>{
                if(response.body["code"] == 0){
                    this.station_info = response.body.attach;
                    console.log(this.station_info);
                }else{
                    console.log("can not find data");
                    this.station_info = [];
                }
            }, response=>{
                console.log("fail");
            });
        }
    }
}
</script>

