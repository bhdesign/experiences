<script setup lang="ts">
import{ref} from 'vue'
import test1 from "./components/test1.vue";
import test2 from "./components/test2.vue";
import test3 from "./components/test3.vue";
let t1=ref(false),t2=ref(false), t3=ref(false)
function displaytest(n:number){
switch (n){
  case 1:
    t1.value=true
    t2.value=false
     t3.value=false
    break
  case 2:
  t1.value=false
  t2.value=true
   t3.value=false
    break
  case 3:
  t1.value=false
  t2.value=false
  t3.value=true
    break
}
}
function hideTest(){
  t1.value=false
    t2.value=false
    t3.value=false
}
</script>

<template>
  <div v-if="t1 || t2 || t3" class="close" @click="hideTest">Close</div>
  <div class="myBtn" v-if="!t1 && !t2 && !t3">
    <div @click="displaytest(1)">Test 1</div>
    <div @click="displaytest(2)">Test 2</div>
    <div @click="displaytest(3)">Test 3</div>
  </div>
  <test1 v-if="t1"/>
  <test2 v-if="t2"/>
  <test3 v-if="t3"/>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
