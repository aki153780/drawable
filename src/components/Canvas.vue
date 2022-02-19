<template>
  <div class="w-full">
    <div class="m-4">
      <button
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 mr-2 rounded"
      >
        送信
      </button>
      <button
        class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 mr-2 rounded"
        @click="deleteAll"
      >
        すべて削除
      </button>
    </div>
    <canvas
      id="myCanvas"
      ref="canvas"
      :width="canvasWidth"
      :height="canvasHeight"
      @mousedown="dragStart"
      @mouseup="dragEnd"
      @mouseout="dragEnd"
      @mousemove="draw"
    ></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";

type Position = {
  x: number;
  y: number;
};

export default defineComponent({
  name: "Canvas",
  props: {
    msg: String,
  },
  setup(props) {
    const pointX = ref<number>(0);
    const canvasRect = ref<DOMRect | null>(null);
    const prevPosition = ref<Position>({ x: 0, y: 0 });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const context = ref<CanvasRenderingContext2D | null>();
    const isDrag = ref(false);
    const canvasWidth = ref(600);
    const canvasHeight = ref(400);

    const draw = (e: MouseEvent): void => {
      if (!isDrag.value) {
        return;
      }
      if (context.value && canvasRect.value) {
        context.value.beginPath();
        context.value.moveTo(prevPosition.value.x, prevPosition.value.y);
        context.value.lineTo(
          e.clientX - canvasRect.value.x,
          e.clientY - canvasRect.value.y
        );
        context.value.stroke();
        prevPosition.value = {
          x: e.clientX - canvasRect.value.x,
          y: e.clientY - canvasRect.value.y,
        };
      }
    };

    const dragStart = (e: MouseEvent): void => {
      drawStart(e.clientX, e.clientY);
    };
    const touchStart = (e: TouchEvent): void => {
      //drawStart(e., e.clientY);
    };
    const drawStart = (x: number, y: number): void => {
      if (!canvasRect.value) return;
      isDrag.value = true;
      prevPosition.value = {
        x: x - canvasRect.value.x,
        y: y - canvasRect.value.y,
      };
    };
    const dragEnd = (e: MouseEvent): void => {
      isDrag.value = false;
    };

    const deleteAll = (): void => {
      context.value?.clearRect(0, 0, canvasWidth.value, canvasHeight.value);
    };

    onMounted(() => {
      context.value = canvas.value?.getContext("2d");
      if (!canvas.value) {
        return;
      }
      canvasRect.value = canvas.value.getBoundingClientRect();
      if (context.value === null || context.value === undefined) return;
      context.value.lineWidth = 5;
      context.value.strokeStyle = "#000000";
      context.value.lineCap = "round";
    });

    return {
      canvasWidth,
      canvasHeight,
      draw,
      dragStart,
      dragEnd,
      deleteAll,
      pointX,
      canvas,
      isDrag,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
#myCanvas {
  border: 1px solid #000000;
}
</style>
