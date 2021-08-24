<template>
    <div> 
        <el-row>
            <el-col :span="6"><el-input v-model="newURL" placeholder="url"></el-input></el-col>
            <el-col :span="2">
                <el-button @click="insert()">插入视频流</el-button>
            </el-col>
            <!-- <el-col :span="2">             -->
                <el-button @click="getvideoList()">刷新视频列表</el-button>
            <!-- </el-col> -->
            <!-- <el-col :span="2">
            </el-col> -->
            <!-- <el-col :span="2"> -->
                <el-button @click="createVideo()">启动视频流</el-button>
            <!-- </el-col> -->
            <!-- <el-col > -->
                <el-button @click="show3 = !show3">选择视频流</el-button>
                <el-collapse-transition>
                    <div v-show="show3" title="视频流选取">
                        <el-transfer 
                        v-model="value" 
                        :data="videoList" 
                        :titles="['未选视频','已选视频']" 
                        @change="handleChange"
                        ></el-transfer>
                    </div>
                </el-collapse-transition>
            <!-- </el-col> -->
        </el-row>
        <br>
        <el-row :gutter="10">
            <el-col :xs="24" :sm="24" :md="12" :lg="8" v-for="item in value" :key="item">
                <video autoplay controls width="100%" class="vjs-big-play-centered"  :id="`videoElement${videoList[item].key}`"></video>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    import flvjs from 'flv.js'
    import axios from 'axios'
    export default {
        data(){
            return {
                value:[],
                videoList : [],
                newURL:"",
                show3:false,
                cnt:0
            }
        },
        name: 'HelloWorld',
        props: {
            msg: String
        },
        mounted() {
            this.$nextTick(() => {
                this.createVideo()
            })
        },
        methods: {
            insert(){
                // console.log(this.videoitem.url)
                if(this.newURL!="")
                {
                    console.log(this.newURL)
                    var tempitem = {
                        key:this.videoList.length,
                        value:{
                            url:this.newURL,
                            port:"",
                            app:"",
                            stream:""
                            },
                        disabled:false,
                        label:`${this.videoList.length}-自选视频流-${this.cnt}`,
                        player:null
                        }
                    this.cnt++
                    console.log(tempitem)
                    this.videoList.push(tempitem)
                    this.newURL=""
                }
            },
            handleChange(value,direction,movedKyes){
                if(direction=='left')
                {
                    for(let i=0;i<movedKyes.length;++i)
                    {
                        let j=movedKyes[i]
                        if(this.videoList[j].player!=null)
                        {
                            this.videoList[j].player.unload()
                            this.videoList[j].player.detachMediaElement()
                            this.videoList[j].player = null
                        }
                    }
                }
            },
            getvideoList(){
                var that = this
                axios.get("http://localhost:5001/api/getallports").then(function(response){
                    that.videoList = []
                    that.cnt = []
                    that.value = []
                    for(let i = 0 ;i<response.data.data.length;++i)
                    {
                        var tempitem = {
                            key:that.videoList.length,
                            value:response.data.data[i],
                            label:`${i}-视频流-${i}`,
                            disabled:false,
                            player:null
                            }
                        that.videoList.push(tempitem)
                    }
                    // console.log(that.videoList)
                    // that.videoList = response.data.data
                },function(error){
                    console.log("not_get_videolist");
                });
            },
            createVideo() {
                if (flvjs.isSupported()) {
                    for(let i=0;i<this.value.length;++i)
                    {
                        let j = this.value[i]
                        
                        var videoElement = document.getElementById(`videoElement${this.videoList[j].key}`);
                        if(this.videoList[j].player != null)
                        {
                            console.log("-------------------------\n-------------------------\n-------------------------\n-------------------------\n-------------------------\n-------------------------\n")
                            this.videoList[j].player.unload()
                            this.videoList[j].player.detachMediaElement()
                            this.videoList[j].player.attachMediaElement(videoElement)
                            this.videoList[j].player.load()
                            this.videoList[j].player.play()
                            continue;
                        }
                        var URL = "";
                        if(this.videoList[j].value.port!="")
                        {
                            URL = "http://"+this.videoList[j].value.url+"/live?";
                            if(this.videoList[j].value.port!="")
                                URL+="port="+this.videoList[j].value.port+"&";
                            if(this.videoList[j].value.app!="")
                                URL+="app="+this.videoList[j].value.app+"&";
                            if(this.videoList[j].value.stream!="")                        
                                URL+="stream="+this.videoList[j].value.stream;
                        }else
                        {
                            URL = this.videoList[j].value.url;                           
                        }
                        console.log(URL)
                        this.videoList[j].player = flvjs.createPlayer({
                            type: 'flv',
                            isLive:true,
                            withCredentials:false,
                            cors:true,
                            url: URL //url地址
                        },{
                            enableWorker: false, //不启用分离线程
                            enableStashBuffer: false, //关闭IO隐藏缓冲区
                            isLive: true,
                            lazyLoad: false,
                        }
                        );
                        this.videoList[j].player.attachMediaElement(videoElement);
                        this.videoList[j].player.load();
                        this.videoList[j].player.play();
                        // console.log(this.videoList[j].player == null)
                        // this.videoList[j].player=flvPlayer
                    }
                }
                //   setInterval(() => {
                //   if (this.flvPlayer.buffered.length) {
                //     let end = this.flvPlayer.buffered.end(0);//获取当前buffered值
                //     let diff = end - this.flvPlayer.currentTime;//获取buffered与currentTime的差值
                //     if (diff >= 0.5) {//如果差值大于等于0.5 手动跳帧
                //       this.flvPlayer.currentTime = this.flvPlayer.buffered.end(0);//手动跳帧
                //   }
                //   }
                // }, 800); //800毫秒执行一次
            }
        }
        
    }


</script>
