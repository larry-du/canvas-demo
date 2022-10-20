<script setup>
import { markRaw, ref } from "vue";
import Test1 from "@/components/Test1.vue";
import Test2 from "@/components/Test2.vue";

const foo = markRaw({ Test1, Test2 });

const current = ref("Test1");
</script>

<template>
  <div class="about">
    <component :is="foo[current]"></component>
    <button @click="current = 'Test1'">111</button>
    <button @click="current = 'Test2'">222</button>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
