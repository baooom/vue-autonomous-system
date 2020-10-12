<template>
    <div>
        <el-card shadow="hover" style="height:300px;">
            这里面显示视频
        </el-card>
        <el-row :gutter="10">
            <el-col :span="12">
            <el-card shadow="hover" style="height:300px;">
                这里面显示车辆当前的速度等信息

            </el-card>
            </el-col>
            <el-col :span="12">
            <el-card shadow="hover" style="height:300px;">
                这里面可以选择网约任务
            </el-card>
            </el-col>
        </el-row>
    </div>
</template>
<script>
export default {
    created(){
        this.init();
    },
    methods:{
        init(){
            //初始化从后台获取数据
            //this.$http.get('localhost:8081/api/v2/parks')
        }
    }
}
</script>

