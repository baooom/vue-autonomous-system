<template>
    <div>
        <el-row :gutter="20">
            <el-col :span="8">
                <el-card shadow="hover" class="mgb20" style="height:252px;">
                    <div class="user-info">
                        <img src="../../assets/img/img.jpg" class="user-avator" alt />
                        <div class="user-info-cont">
                            <div class="user-info-name">{{name}}</div>
                            <div>{{role}}</div>
                        </div>
                    </div>
                    <div class="user-info-list">
                        上次登录时间：
                        <span>2020-10-06</span>
                    </div>
                    <div class="user-info-list">
                        上次登录地点：
                        <span>杭州</span>
                    </div>
                </el-card>
                <el-card shadow="hover" style="height:600px;">
                    <div slot="header" class="clearfix">
                        <span>园区车辆在线情况</span>
                    </div>
                    <div v-for="(item, index) in parks">{{item["name"]}}
                        <el-progress :percentage="0.0" color="#42b983"></el-progress>
                    </div>
                </el-card>
            </el-col>
            <el-col :span="16">
                <el-row :gutter="20" class="mgb20">
                    <el-col :span="8">
                        <el-card shadow="hover" :body-style="{padding: '0px'}">
                            <div class="grid-content grid-con-1">
                                <i class="el-icon-lx-people grid-con-icon"></i>
                                <div class="grid-cont-right">
                                    <div id="connected_car" class="grid-num">{{connected_car.length}}</div>
                                    <div>在线车辆数量</div>
                                </div>
                            </div>
                        </el-card>
                    </el-col>
                    <el-col :span="8">
                        <el-card shadow="hover" :body-style="{padding: '0px'}">
                            <div class="grid-content grid-con-2">
                                <i class="el-icon-lx-notice grid-con-icon"></i>
                                <div class="grid-cont-right">
                                    <div class="grid-num" >{{parks.length}}</div>
                                    <div>园区数量</div>
                                </div>
                            </div>
                        </el-card>
                    </el-col>
                    <el-col :span="8">
                        <el-card shadow="hover" :body-style="{padding: '0px'}">
                            <div class="grid-content grid-con-3">
                                <i class="el-icon-lx-goods grid-con-icon"></i>
                                <div class="grid-cont-right">
                                    <div class="grid-num">100</div>
                                    <div>历史订单数量</div>
                                </div>
                            </div>
                        </el-card>
                    </el-col>
                </el-row>
                <el-card shadow="hover" style="height:403px;">
                    <div slot="header" class="clearfix">
                        <span>车辆情况</span>
                        <el-button style="float: right; padding: 3px 0" type="text">添加</el-button>
                    </div>
                    <el-table :show-header="false" :data="cars_status" style="width:100%;">
                        <el-table-column>
                            <template slot-scope="scope">
                                <div
                                    class="todo-item"
                                >车辆VIN: {{scope.row.vin}}  x: {{scope.row.x}}  y: {{scope.row.y}} z: {{scope.row.z}}</div>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-card>
                <el-card shadow="hover" style="height:403px;">
                     <div class="schart-box">
                        <div class="content-title">统计图</div>
                        <schart class="schart" canvasId="ring" :options="options4"></schart>
                    </div>
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
Vue.use(VueResource);

export default {
    name: 'dashboard',
    data() {
        return {
            name: localStorage.getItem('ms_username'),
            options4: {
                type: 'ring',
                title: {
                    text: '车辆情况'
                },
                showValue: false,
                legend: {
                    position: 'bottom',
                    bottom: 40
                },
                bgColor: '#fbfbfb',
                labels: ['在线车辆', '离线车辆'],
                datasets: [
                    {
                        data: [500, 500]
                    }
                ]
            },
            connected_car:[],
            parks:[],
            cars_status:[],
            url:'10.192.153.229:8080'
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
            setInterval(this.fetch, 1000);
        })
    },
    methods: {
        changeDate() {
            const now = new Date().getTime();
            this.data.forEach((item, index) => {
                const date = new Date(now - (6 - index) * 86400000);
                item.name = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
            });
        },
        fetch(){
            console.log('fetch data');
            this.$http.get('http://'+this.url+'/api/v2/info/car/connected').then(response=>{
                if(response.body["code"] == 0){
                    this.connected_car = response.body.attach;
                }else{
                    this.connected_car = [];
                }
            }, response=>{
                console.log("fail");
            });

            this.$http.get('http://'+this.url+'/api/v2/parks').then(response=>{
                if(response.body["code"] == 0){
                    this.parks = response.body.attach;
                }else{
                    this.parks = [];
                }
            }, response=>{
                console.log("fail");
            });
            // {
            //         params:{vin:this.connected_car[i]}
            // }
            var tmp_list = [];
            var flag = false;
            for(var i=0;i<this.connected_car.length;i++){
                if(i==this.connected_car.length-1){
                    flag = true;
                }
                this.$http.get('http://'+this.url+'/api/v2/info/car/' + this.connected_car[i]  + '/position').then((function(vin,flag){
                    return function(response){
                        //vin对应的位置信息
                        if(response.body["code"] == 0){
                            var tmp = response.body.attach;
                            tmp['vin'] = vin; 
                            tmp_list.push(tmp);
                            if(flag == true){
                                this.cars_status = tmp_list;
                            }
                        }else{
                            this.cars_status = [];
                        }
                        
                    }
                })(this.connected_car[i],flag) , response=>{
                    
                    console.log("fail");
                });
            }

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
