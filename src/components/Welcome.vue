<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
    </el-breadcrumb>
    <h3>Welcome, {{username}}</h3>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        username:''
      }
    },
    created(){
      this.getUsername()
    },
    methods:{
      getUsername(){
       const username = window.sessionStorage.getItem('username');
       this.username = username;
      }
    }
  }
</script>
