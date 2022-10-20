<script setup>
import { ref, onMounted } from "vue";

const printArea = ref(null);
const initMouseX = ref(0);
const initMouseY = ref(0);
const isDraw = ref(false);
const ctx = ref(null);
const currentColor = ref("black");
const colorChoose = ref([
  { title: "black", type: "black" },
  { title: "red", type: "red" },
  { title: "blue", type: "lightblue" },
  { title: "green", type: "lightgreen" },
]);
const strokeWidth = ref([
  { type: 8 },
  { type: 16 },
  { type: 24 },
  { type: 32 },
]);

onMounted(() => {
  initCanvas();
});

const drawStart = (event) => {
  initMouseX.value = event.clientX;
  initMouseY.value = event.clientY;
  isDraw.value = true;
  ctx.value.beginPath();
  ctx.value.moveTo(initMouseX.value, initMouseY.value);
};

const onDraw = (event) => {
  if (!isDraw.value) return;
  ctx.value.lineCap = "round"; //設定畫筆端點為圓的
  ctx.value.lineJoin = "round"; //設定畫筆轉彎處為圓的
  ctx.value.lineTo(event.clientX, event.clientY);
  ctx.value.stroke();
};

const drawEnd = (event) => {
  isDraw.value = false;
};

const initCanvas = () => {
  printArea.value.width = window.innerWidth;
  printArea.value.height = window.innerHeight - 50;
  ctx.value = printArea.value.getContext("2d");
};

const test = ref(16);
</script>

<template>
  <section class="canvas-view">
    <canvas
      id="print-area"
      ref="printArea"
      @mousedown="drawStart"
      @mousemove="onDraw"
      @mouseup="drawEnd"
    ></canvas>

    <div class="style-edit">
      <pre>{{ test }}</pre>
      <div class="stroke-width-choose">
        <button
          class="color-button"
          v-for="currentWidth in strokeWidth"
          :key="currentWidth.type"
          @click="
            (ctx.lineWidth = currentWidth.type), (test = currentWidth.type)
          "
        >
          {{ currentWidth.type }}
        </button>
      </div>
      <div class="color-choose">
        <button
          class="color-button"
          @click="ctx.strokeStyle = 'white'"
        ></button>
        <button
          :style="{ 'background-color': colorType.title }"
          class="color-button"
          v-for="colorType in colorChoose"
          :key="colorType.title"
          @click="ctx.strokeStyle = colorType.type"
        ></button>
      </div>
    </div>
  </section>
</template>

<style lang="sass" scoped>
.canvas-view
  position: relative
.style-edit
  position: absolute
  right: 0
  left: 0
  bottom: 0
  text-align: center
.color-button
  width: 100px
  height: 100px
  border-radius: 50%
  border: 1ps solid
</style>
