<script setup>
import { ref, onMounted } from "vue";

const printColor = ref([
  { type: "black" },
  { type: "red" },
  { type: "lightblue" },
  { type: "lightgreen" },
]);

const canvasContext = ref(null);
const printArea = ref(null);

onMounted(() => {
  initCanvas();
});

const initCanvas = () => {
  printArea.value.width = window.innerWidth;
  printArea.value.height = window.innerHeight - 50;
  canvasContext.value = printArea.value.getContext("2d");
};

const isDraw = ref(false);

const drawStart = (positionX, positionY) => {
  isDraw.value = true;
  canvasContext.value.beginPath();
  canvasContext.value.moveTo(positionX, positionY);
};
const drawOn = (positionX, positionY) => {
  if (!isDraw.value) return;
  canvasContext.value.lineCap = "round";
  canvasContext.value.lineJoin = "round";
  canvasContext.value.lineTo(positionX, positionY);
  canvasContext.value.stroke();
};

const drawEnd = () => {
  isDraw.value = false;
  saveImageStep();

};

const saveImageStep = () => {
  const printBase64 = printArea.value.toDataURL();
  stepRecord.value = [...stepRecord.value, printBase64];
  drawStep.value++;
}

const clearCurrentImage = () => {
  canvasContext.value.clearRect(
    0,
    0,
    printArea.value.width,
    printArea.value.height
  );
};

const drawStep = ref(-1);
const stepRecord = ref([]);

const drawUndo = () => {
  const isInitImage = drawStep.value <= 0;
  if (!isInitImage) {
    drawStep.value--;
    drawCurrent();
  } else {
    drawStep.value = -1;
    clearCurrentImage();
  }
};
const drawRedo = () => {
  const isLatestImage = drawStep.value >= stepRecord.value.length - 1;
  if (isLatestImage) return;
  drawStep.value++;
  drawCurrent();
};

const drawCurrent = () => {
  const canvasImage = new Image();
  canvasImage.src = stepRecord.value[drawStep.value];
  canvasImage.onload = () => {
    clearCurrentImage();
    canvasContext.value.drawImage(canvasImage, 0, 0);
  };
};

const strokeWidth = ref(1);
const currentColor = ref("black");

const changeStrokeWidth = (currentStrokeWidth) => {
  strokeWidth.value = currentStrokeWidth;
  canvasContext.value.lineWidth = currentStrokeWidth;
};

const changeStrokeColor = (strokeColor) => {
  canvasContext.value.strokeStyle = strokeColor;
  currentColor.value = strokeColor;
};
</script>

<template>
  <section class="canvas-view">
    <canvas
      id="print-area"
      ref="printArea"
      @mousedown="drawStart($event.clientX, $event.clientY)"
      @mousemove="drawOn($event.clientX, $event.clientY)"
      @mouseup="drawEnd"
      @touchstart="
        drawStart($event.touches[0].screenX, $event.touches[0].screenY)
      "
      @touchmove.prevent="
        drawOn($event.touches[0].screenX, $event.touches[0].screenY)
      "
      @touchend="drawEnd"
    ></canvas>

    <div class="style-edit">
      <div>current stroke width: {{ strokeWidth }}px</div>
      <input
        type="range"
        id="stroke-width"
        name="stroke-width"
        class="stroke-width"
        step="1"
        min="1"
        max="32"
        :value="strokeWidth"
        @input="changeStrokeWidth($event.target.value)"
      />

      <div class="color-choose">
        <button @click.prevent="clearCurrentImage">Clear All</button>
        <button @click.prevent="canvasContext.strokeStyle = 'white'">
          Eraser
        </button>
        <button
          :style="{ 'background-color': colorType.type }"
          class="color-button"
          :class="{ 'color-active': currentColor === colorType.type }"
          v-for="colorType in printColor"
          :key="colorType.type"
          @click.prevent="changeStrokeColor(colorType.type)"
        ></button>
        <button @click.prevent="drawUndo">Undo</button>
        <button @click.prevent="drawRedo">Redo</button>
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

.color-choose
  button + button
    margin-left: 5px
.color-button
  width: 50px
  height: 50px
  border-radius: 50%
  border: 1px solid
.color-active
  box-shadow: 0 0 5px 5px inset white
</style>
