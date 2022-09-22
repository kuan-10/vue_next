<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import {ref} from 'vue'
const message=ref(2)
console.log(message)
function add(){
  this.message=3
}
</script>

<template>
 <div @click="add()">{{message}}</div>
</template>

<style scoped>
div{
  width: 200px;
  height: 200px;
}
</style>
