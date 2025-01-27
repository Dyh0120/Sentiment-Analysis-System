<template>
    <div :class="container">
      <form v-if="isPc" class="pc">
        <div class="form_left">
          <h1 style="font-size: 32px;">欢迎登录</h1>
          <!-- <h3>账号登录</h3> -->
          <div class="uname">
            <div>
              <el-icon :size="24"><UserFilled /></el-icon>
            </div>
            <input type="text" placeholder="学号/工号/账号" v-model="uname" />
          </div>
          <div class="password">
            <div>
              <el-icon :size="24"><Lock /></el-icon>
            </div>
            <input type="password" placeholder="密码" v-model="password" />
          </div>
          <div class="identify">
            <input type="text" placeholder="验证码" v-model="cap">
            <span>
                <img :src="imageSrc" alt="Received Image" height="75%" @click="changecap"/>
            </span>
          </div>
          <div class="btn"><button class="log" @click.prevent="log">登录</button></div>
        </div>
      </form>
      <form v-if="isPc===false" class="mobile">
          <h1>资助统一身份认证</h1>
          <!-- <h3>账号登录</h3> -->
          <div class="uname">
            <div>
              <el-icon :size="24"><UserFilled/></el-icon>
            </div>
            <input type="text" placeholder="学号" v-model="uname" />
          </div>
          <div class="password">
            <div>
              <el-icon :size="24"><UserFilled /></el-icon>
            </div>
            <input type="password" placeholder="密码" v-model="password" />
          </div>
          <div class="remenber">
            <el-checkbox v-model="remenber" label="记住密码" size="large" fill="#f5f5f5" />
          </div>
          <div class="btn"><button class="log" @click.prevent="log">登录</button></div>
      </form>
    </div>
</template>

<script setup>
import { onMounted, ref,watch } from 'vue'
import axios from 'axios'
import { ElForm, ElFormItem, ElInput, ElButton, ElCheckbox, ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'
import { UserFilled,Lock } from '@element-plus/icons-vue'
import isLogin from '@/api/isLogin'
// 验证码
const uuid=ref()
const imageSrc = ref(null);
let cap=ref()
    const fetchImage = async () => {
      try {
        const response = await fetch('http://mxt.nat300.top/cap/');
        uuid.value=response.headers.get('uuid')
        // console.log(response.headers.get('uuid'));
        // console.log(response.headers.get('content-type'));

        const blob = await response.blob();
        // 将 Blob 转换为 Data URL
       const dataUrl = URL.createObjectURL(blob);
        // 设置图片的 src 属性
        imageSrc.value = dataUrl;
      } catch (error) {
        console.error('Error fetching image:', error);
      }
    };
const changecap=()=>{
  fetchImage()
}
let container="container container1"
const router = useRouter()
// 表单数据和状态
const uname = ref('')
const password = ref('')
const loading = ref(false)
// const clientId = ref('')
const token=localStorage.getItem('token');
const role =localStorage.getItem('role')
const redirect = router.currentRoute.value.query.redirect
// 定义一个响应式变量来存储客户端ID
const clientId = ref('');
// 定义一个变量来判断是电脑端还是移动端
let isPc = true;
// 定义一个函数来判断是否是电脑端
const isPC = () => {
  const userAgent = navigator.userAgent;

  // 定义一些常见的移动设备和浏览器的用户代理特征
  const mobileAgents = [
    /android/i,     // Android设备
    /iphone|ipad|ipod/i, // iOS设备
    /windows phone/i, // Windows Phone设备
    /blackberry/i,  // Blackberry设备
    /opera mini/i,  // Opera Mini浏览器（通常用于移动设备）
    /mobile/i,      // 通用移动设备标记
    /touch/i        // 触摸设备标记（可能包括桌面触摸屏）
  ];

  // 初始化isPc为true
  isPc = true;

  // 检查用户代理字符串是否包含任何移动设备的特征
  for (let i = 0; i < mobileAgents.length; i++) {
    if (mobileAgents[i].test(userAgent)) {
      isPc = false; // 如果是移动设备，则将isPc设置为false
    }
  }
  // console.log(isPc);
};


onMounted(
async() => {
  fetchImage()
   isPC()
   if(isPc===false)
      container="container container2"
   else
      container="container container1"
 const isLoggedin=await isLogin();
  if(!isLoggedin)
  {
    localStorage.removeItem('role')
    localStorage.removeItem('token')
  }


  if(redirect&&token&&role)
    {
      console.log('redirect', redirect)
       window.location.href = `${redirect}?token=${token}&role=${role}`
    }
})



// watch(clientId, (newValue, oldValue) => {
//   // location.reload()
// });

const log = async () => {
  try {
    loading.value = true
    const config = {
      headers: {
    'content-language': 'zh_CN'
  }
};
    const response = await axios.post('http://mxt.nat300.top/user/login/', {
      username: uname.value,
      password: password.value,
      uuid:uuid.value,
      answer:cap.value
    }, config)
console.log(response.data);


    if (response.data.code == 0) {
      ElMessage.success('登录成功')
      localStorage.setItem('token', response.data.data.token)
      // console.log('response.data是', response.data.data.roles[0].roleName)
      //判断是否有回调参数
      localStorage.setItem('role', response.data.data.role)
    }
    else {
      ElMessage.error(response.data.message)
    }
  } catch (error) {
    loading.value = false
    ElMessage.error('登录失败，请稍后重试')
  }
}
</script>

<style scoped>
input {
  -webkit-box-shadow: 0 0 0 1000px #f5f5f5 inset;
  border: 0;
  outline: none;
}

* {
  padding: 0;
  margin: 0;
  /* 改盒子高度和长度的计算 */
  box-sizing: border-box;
}

img {
  /* 除掉图片底端的空隙 */
  border: 0;
  /* 图片和文字中心对齐 */
  vertical-align: middle;
}

button {
  /* 让鼠标变小手 */
  cursor: pointer;
}

p,
h1,
h2,
h3,
h4,
h5,
span,
form {
  cursor: default;
}

button,
input {
  font-family: 微软雅黑;
}
input[type='text']:focus {
  background-color: #f5f5f5;
}
.clearfix::after {
  visibility: hidden;
  clear: both;
  display: block;
  content: '';
  height: 0;
}

.container {
  width: 100vw;
  height: 100vh;

  display: flex;
  /* 交叉轴：竖直 */
  align-items: center;
  /* 主轴水平 */
  justify-content: center;
}
.container1{
   /*background: url(./imag/b3925630992729172938c08655e5cfd0.jpg) no-repeat;*/
   background-color: #d2e7ee;
   background-size: cover;
   width: 100vw;
  height: 100vh;
}
.container2{
   background: url(./../../../image/mobile登录背景.png) fixed no-repeat;
   background-size: cover;
   width: 100vw;
  height: 100vh;
}
.container::-webkit-scrollbar {
  display: none !important;
}
.container .pc {
  width: 430px;
  height: 460px;
  backdrop-filter: blur(80px);
  background-color: #fff;
  border-radius: 5px;
}
.container .pc .form_left {
  width: 100%;
  height: 100%;
  float: right;
  margin-top: 30px;
}
.container .pc .form_left h1 {
  font-size: 24px;
  text-align: center;
  letter-spacing: 0.05em;
}
.container .pc .form_left h3 {
  font-size: 20px;
  text-align: center;
  padding-top: 15px;
  letter-spacing: 0.05em;
}

.container .uname,
.password,.identify {
  background-color: #f5f5f5;
  width: 70%;
  height: 12%;
  margin: 5% auto;
  padding: 0;
  border-radius: 5px;
  vertical-align: middle;
}

.el-checkbox.el-checkbox--large {
  height: 24px;
}
.container .pc .form_left .uname input,
.password input{
  margin-left: 4px;
  font-size: 16px;
  width: 80%;
  border: none;
  outline: none;
  background-color: #f5f5f5;
}
.identify input{
  font-size: 16px;
  width: 53%;
  border: none;
  outline: none;
  background-color: #f5f5f5;
  margin: 5.5%;
}
input:focus {
  background-color: #f5f5f5 !important;
}
.container .pc .form_left .uname span,
.password span {
  height: 120%;
  width: 40%;
}
.btn {
  margin: auto;
  width: 70%;
  height: 12%;
  margin: 8% auto;
}
.log {
  width: 100%;
  height: 120%;
  background-color: #4a80d1;
  border-radius: 5px;
  color: #fff;
  letter-spacing: 0.05em;
  outline: none;
  border: none;
  font-size: 16px;
}
.uname div,
.password div {
  display: inline-block;
  width: 15%;
  vertical-align: middle;
  padding: 15px;
}

/* 移动端*/
.mobile{
  background-color: #fff;
  border-radius: 10px;
  width: 250px;
  height: 280px;
}
.mobile h1{
  font-size: 22px;
  text-align: center;
  line-height: 24px;
  padding: 20px 0;
}
.mobile .uname div,.mobile .password div{
  padding: 5px;
}
.mobile .uname input,.mobile .password input {
  margin-left: 10px;
  width: 135px;
}
.mobile .btn{
  margin-top: 35px;
}
</style>
