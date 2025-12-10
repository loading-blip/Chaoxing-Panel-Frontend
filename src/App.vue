<script setup>
import ActivityList from './components/ActivityList.vue'
import DefaultCompose from './components/DefaultComponent.vue'
import Transcript from './components/Transcript.vue'
import {shallowRef} from 'vue'

const current_component = shallowRef(DefaultCompose)

const function_map=[
    {
    id:0,
    name:"default",
    compose: DefaultCompose
  },
  {
    id:1,
    name:"activity",
    compose:ActivityList
  }

]

</script>


<template>
  <div class="common-layout" >
    <el-container>
      <el-header>
        <el-row>
          <el-col :span="8">
            <!-- <div class="grid-content ep-bg-purple" /> -->
          </el-col>
          <el-col :span="8">
            <h1 class="app_name">第二课堂</h1>
          </el-col>
          <el-col :span="8">
            <!-- <div class="grid-content ep-bg-purple" /> -->
          </el-col> 
        </el-row>
      </el-header>
      <el-container>
        <el-aside width="200px">
          <el-scrollbar>
            <el-menu :default-openeds="['1']">
              <el-sub-menu index="1">
                <template #title>
                  <el-icon><message /></el-icon>信息
                </template>
                <el-menu-item index="1-1" @click="current_component = ActivityList">活动列表</el-menu-item>
                <el-menu-item index="1-2" @click="current_component = Transcript">成绩单</el-menu-item>
              </el-sub-menu>
            </el-menu>
          </el-scrollbar>
        </el-aside>
        <el-main>
          <keep-alive>
            <component :is="current_component"/>
          </keep-alive>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<style scoped>
.app_name{
  text-align: center;
  margin: 0;
}
header{
  border-bottom: 1px solid #505050;
}
main{
  padding: 10px 0 0 10px;
}
/* aside{
  box-shadow: 3px 3px;
} */
</style>

