<template>
  <div>
    <a href="#" class="btn">Welcome</a>
    <div class="icon1">⚡</div>
    <div class="class1">用于学习的一个电商后台项目，本项目主要有vuejs、element-ui、axios、echarts等技术栈</div>
    
  </div>
</template>

<script>
export default {
  created() {
    setTimeout(()=>{
       this.$message.success('欢迎来到XXX电商管理系统~ ^-^')
    },1000)
    //  this.$message.success('欢迎来到XXX电商管理系统~ ^-^')
  }
}
</script>

<style lang="less" scoped>
   .btn {
      position: absolute;
      top: 20%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 230px;
      height: 90px;
      line-height: 90px;
      text-align: center;
      color: #fff;
      font-size: 24px;
      text-transform: uppercase;
      text-decoration: none;
      font-family: sans-serif;
      box-sizing: border-box;
      background: linear-gradient(90deg, #0ce24d, #f441a5, #0a66df, #da7a21);
      background-size: 400%;
      border-radius: 60px;
      z-index: 1;
    }
    
    .btn:hover {
      animation: animate 8s linear infinite;
    }
    
    @keyframes animate {
      0% {
        background-position: 0%;
      }
      100% {
        background-position: 400%;
      }
    }
    
    .btn::before {
      content: '';
      position: absolute;
      top: -5px;
      left: -5px;
      right: -5px;
      bottom: -5px;
      z-index: -1;
      background: linear-gradient(90deg, #03a9f4, #f441a5, #ffeb3b, #03a9f4);
      background-size: 400%;
      border-radius: 40px;
      opacity: 0;
      transition: 0.5s;
    }
    
    .btn:hover::before {
      filter: blur(20px);
      opacity: 1;
      animation: animate 8s linear infinite;
    }
     .icon1 {
       position: absolute;
       top: 40%;
       left: 50%;
       transform: translate(-50%,-50%);
        width: 100px;
        height: 100px;
        border-radius: 50%;
        box-shadow: 0 0 30px rgb(13, 219, 92);
        background-color: #fff;
        display: inline-block;
        text-align: center;
        line-height: 100px;
        font-size: 40px;
        animation: animate1 2s infinite;
  }
  @keyframes animate1 {
    0%{box-shadow: 0 0 15px rgb(13, 219, 92);}
    25%{box-shadow: 0 0 15px rgb(44, 16, 204);}
    50%{box-shadow: 0 0 15px rgb(94, 3, 3);}
    75%{box-shadow: 0 0 15px rgb(219, 13, 126);}
    100%{box-shadow: 0 0 15px rgb(226, 197, 34);}
  }
  .class1 {
     position: absolute;
       top: 60%;
       left: 50%;
       transform: translate(-50%,-50%);
       font-size: 20px;
       font-weight: 700;
       color: #000
  }
</style>