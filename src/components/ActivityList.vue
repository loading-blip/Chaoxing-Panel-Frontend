<script setup>
import { tr } from 'element-plus/es/locales.mjs';
import { unique } from 'element-plus/es/utils/arrays.mjs';
import {ref,onMounted} from 'vue'

const t_head = ref([])
const t_body = ref([])
const act_status = ref([])

const is_ready = ref(false)
const current_work = ref("")
const current_quantity = ref(0)
const quantity = ref(0)

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

function set_data(rep_json){
    rep_json = JSON.parse(rep_json)
    if (rep_json["code"] === "202"){
        throw new Error("数据还在获取中")
    }
    let datas = rep_json["data"]
    for (let data of datas){
        t_head.value += Object.keys(data)
        act_status.value.push(data["act_status"])
        delete data["act_status"]
        t_body.value.push(data)
    }
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


const backend = setInterval(() => {
    get_backend_status().then((status_json)=>{
        status_json = JSON.parse(status_json)
        console.log(status_json["complete"])
        is_ready.value = status_json["complete"]
        current_work.value = status_json["current_work"]
        current_quantity.value = status_json["current_quantity"]
        quantity.value = status_json["quantity"]
    })
    
    if (is_ready.value){
        clearInterval(backend)
        get_data().then((rep_json)=>set_data(rep_json))
    }
}, 500);

t_head.value = unique(t_head.value)
</script>

<template>
  <el-table v-loading="!is_ready && {text:`${current_quantity}/${quantity}正在获取${current_work}...`}" :data="t_body" stripe style="width: 100%">
    <el-table-column prop="名称" label="名称"/>
    <el-table-column prop="人数" label="人数"/>
    <el-table-column prop="位置" label="位置"/>
    <el-table-column prop="报名开始时间" label="报名开始时间"/>
    <el-table-column prop="报名结束时间" label="报名结束时间"/>
    <el-table-column prop="活动开始时间" label="活动开始时间"/>
    <el-table-column prop="活动结束时间" label="活动结束时间"/>
    <el-table-column prop="描述" label="描述"/>
  </el-table>
</template>

<style scoped lang="scss">
// table {
//     border-collapse: collapse;
//     border-bottom: 1px solid #808080;
//     border-spacing: 0;
//     tr:nth-child(even){
//         background-color: #80808020;
//     }
//     tr:hover{
//         background-color: #80808040;
//     }

//     tr{
//         td:hover{
//             background-color: #80808080;
//         }
//     }
// }
</style>