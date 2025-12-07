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

async function get_session(){
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

async function set_data(status){
    let signal = status
    await get_data().then((rep_json)=>{
                        rep_json = JSON.parse(rep_json)     
                        if (rep_json["code"] === 202){
                            console.log('获取数据中...')
                            signal = false
                        }
                        tables.value = rep_json["data"]
                    })
    console.log(signal)
    return signal
    
}

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
//"current_work":"xxxx","current_quantity":0,"quantity":0,"complete":false

const tableRowClassName = ({row,}) => {
    return row['status']
}

async function service(status){
    let signal = true
    await set_data(status).then((i)=>{signal=i})
    console.log(signal)
    if (signal){
        ElMessage.primary('已刷新')
        return 
    }
    get_session().then(async (response)=>{
        let rep_json = JSON.parse(response)
        console.log(rep_json)
        if (rep_json["code"] === 200){
            ElMessage.primary('成功！')
            is_ready.value = false

            if (!status){
                await sleep(1000);
            }

            const backend = setInterval(() => {
                get_backend_status().then((status_json)=>{
                    status_json = JSON.parse(status_json)
                    is_ready.value = status_json["complete"]
                    current_work.value = status_json["current_work"]
                    current_quantity.value = status_json["current_quantity"]
                    quantity.value = status_json["quantity"]
                })
                
                

                if (is_ready.value){
                    clearInterval(backend)
                    set_data()
                }
            }, 500);
        }
        else{
            ElMessage.error('内部错误')
        }
    })
}

// t_head.value = unique(t_head.value)
</script>

<template>
    <div class="button_group">
        <el-button type="primary" plain @click="service(true)">更新</el-button>
        <el-button type="info" plain @click="service(false)">重新获取</el-button>
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
    }

</style>