<template>
<div class="contain">
  <div class="w">
  <Top/>
  <div class="nav">
    <el-radio-group v-model="isCollapse" style="margin-bottom: 20px">
    <el-radio-button :value="false" size="large" style="margin-left: 10px;">详情</el-radio-button>
    <el-radio-button :value="true" size="large">折叠</el-radio-button>
  </el-radio-group>
  <el-menu
    default-active="2"
    class="el-menu-vertical-demo"
    :collapse="isCollapse"
    @open="handleOpen"
    @close="handleClose"
  >
    <el-menu-item index="1" style="font-size: 17px;letter-spacing: 0.1em;" class="elitem">
      <el-icon :size=38><ChatDotRound /></el-icon>
      <template #title>情感实时分析</template>
    </el-menu-item>
    <el-menu-item index="2" style="font-size: 17px;letter-spacing: 0.1em;" class="elitem">
      <el-icon :size=38><DataLine /></el-icon>
      <template #title>情感状态折线图</template>
    </el-menu-item>
    <el-menu-item index="3" style="font-size: 17px;letter-spacing: 0.1em;" class="elitem">
      <el-icon :size=38><FolderOpened /></el-icon>
      <template #title>使用记录</template>
    </el-menu-item>
    <el-menu-item index="4" style="font-size: 17px;letter-spacing: 0.1em;" class="elitem">
      <el-icon :size=38><User /></el-icon>
      <template #title>个人信息</template>
    </el-menu-item>
  </el-menu>
  </div>
  <div class="show">
    <router-view/>
  </div>
</div>
</div>
</template>

<script lang="ts" setup>
import { DataLine,ChatDotRound,FolderOpened,User } from '@element-plus/icons-vue'
import { ref,watch } from 'vue'
import  Top  from '@/components/Top/index.vue'
import { useRoute, useRouter } from 'vue-router';
import { log } from 'console';
const router = useRouter();
const isCollapse = ref(false)
const index =ref('1')
const handleOpen = (key: string, keyPath: string[]) => {
  console.log(key,keyPath);
  index.value=key
}
const handleClose = (key: string, keyPath: string[]) => {
  console.log(key,keyPath);
  index.value=key
}
watch(index, (newValue, oldValue) => {
  if(newValue==='1') router.push({ path: '/system/analysis'})
  if(newValue==='2') router.push({ path: '/system/linechart'})
  if(newValue==='3') router.push({ path: '/system/record'})
  if(newValue==='4') router.push({ path: '/system/userinfo'})
});
</script>

<style>
* {
  padding: 0;
  margin: 0;
  /* 改盒子高度和长度的计算 */
  box-sizing: border-box;
}
.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 200px;
  min-height: 600px;
}
body{
   background-color: #d2e7ee;

}
.w{
  width: 1200px;
  margin: 0 auto;
  background-color: #fff;
}
.el-menu-item{
  padding: 20px
}
.elitem{
  padding-top: 45px;
  padding-bottom: 45px;
}
.nav{
  width: 200px;
  float: left;
  background-color: #fff;
  height: 660px;
}
.show{
  width: 1000px;
  height: 650px;
  float: left;

}
.contain{
  width: 100vw;
  height: 100vh;

  display: flex;
  /* 交叉轴：竖直 */
  align-items: center;
  /* 主轴水平 */
  justify-content: center;
}
</style>
