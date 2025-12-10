<script setup>
import { ElMessage } from 'element-plus'
import {ref,onMounted} from 'vue'

const tables = ref([])

const is_ready = ref(true)
const current_work = ref("")
const current_quantity = ref(0)
const quantity = ref(0)

function sleep(ms) {
return new Promise(resolve => setTimeout(resolve, ms));
}

// 获取数据
async function get_data()
{
    const api = "http://localhost:5000/data"
    try{
        const response = await fetch(api,{
        method: "GET",
        headers: {"Content-Type": "application/json"},
        timeout: 5000,
        })
        return response.json()
    }catch(error){
        if (error.name === 'AbortError') {
            console.error('请求超时');
        }
        throw error;
    }
}
// 发送session请求
async function post_session(){
    const api = "http://localhost:5000/service"
    const data = JSON.stringify({'use':'get_activity'})
    try{
        const response = await fetch(api,{
        method: "POST",
        headers: {"Content-Type": "application/json"},
        timeout: 5000,
        body: data
        })
        return response.json()
    }catch(error){
        if (error.name === 'AbortError') {
            console.error('请求错误');
        }
        throw error;
    }
}

// 获取状态
async function get_backend_status() {
    const api = "http://localhost:5000/status"
    try{
        const response = await fetch(api,{
            method: "GET",
            headers: {"Content-Type": "application/json"},
        })
        return response.json()
    }catch{
        if (error.name === 'AbortError') {
            console.error('请求超时');
        }
        throw error;
    }
}

//颜色管理
const tableRowClassName = ({row,}) => {
    return row['status']
}

async function services(params) {
    const status_json = JSON.parse(await get_backend_status())
    if (status_json["session"] !== "" && (status_json["status"] === "working" && status_json["session"] !== "get_activity")){
        ElMessage.warning('后端正在处理其他任务')
        return
    }
    else if (params === 'refresh' && status_json["complete"] && status_json["session"]==="get_activity"){
        ElMessage.primary('已从后端缓存中获取数据')
        is_ready.value = false
        const table_data = JSON.parse(await get_data())
        tables.value = table_data["data"]
        is_ready.value = true
        return
    }
    const req = JSON.parse(await post_session())
    if (req['code']!==200){
        ElMessage.error('后端错误')
        throw error('后端错误')
    }
    ElMessage.primary('正在获取数据...')
    is_ready.value = false
    const Interval = setInterval(async ()=>{
        const status_json = JSON.parse(await get_backend_status())
        current_work.value = status_json["current_work"]
        current_quantity.value = status_json["current_quantity"]
        quantity.value = status_json["quantity"]
        
        if (status_json["complete"]){
            const result = JSON.parse(await get_data())
            tables.value = result["data"]
            is_ready.value = true
            clearInterval(Interval)
        }
    },500)
    
}


</script>

<template>
    <div class="button_group">
        <el-button type="primary" plain @click="services('refresh')">更新</el-button>
        <el-button type="info" plain @click="services('update')">重新获取</el-button>
    </div>
  <el-table 
    v-loading="!is_ready && {text:`${current_quantity}/${quantity}正在获取${current_work}...`}"
    style="width: 100%"
    highlight-current-row
    :data="tables" 
    :row-class-name="tableRowClassName">
    <el-table-column prop="名称" label="名称" width="300px"/>
    <el-table-column prop="人数" label="人数" width="100px" />
    <el-table-column prop="位置" label="位置"/>
    <el-table-column prop="报名开始时间" label="报名开始时间"/>
    <el-table-column prop="报名结束时间" label="报名结束时间"/>
    <el-table-column prop="活动开始时间" label="活动开始时间"/>
    <el-table-column prop="活动结束时间" label="活动结束时间"/>
    <el-table-column prop="描述" label="描述" width="250px"/>
  </el-table>
</template>

<style lang="scss">
    .el-table{
        .unavailable{
            --el-table-tr-bg-color: #80808020;
        }
        .not_started{
            --el-table-tr-bg-color: #0000ffa0;
        }
        .full{
            --el-table-tr-bg-color: #ff0000a0;
        }
        .busy{
            --el-table-tr-bg-color: #ffff00a0;
        }
        .great_chance{
            --el-table-tr-bg-color: #008000a0;
        }
        .regd{
            --el-table-tr-bg-color: #5c815ca0;
        }
    }

</style>