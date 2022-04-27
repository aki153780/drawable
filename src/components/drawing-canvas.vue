<template>
  <div class="w-full">
    <div v-if="drawable" class="m-4">
      <button
        class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 mr-2 rounded"
        @click="deleteAll"
      >
        すべて削除
      </button>
    </div>
    <div class="flex justify-center">
      <canvas
        class="drawable-canvas"
        ref="canvas"
        :width="canvasWidth"
        :height="canvasHeight"
        @mousedown="dragStart"
        @touchstart="touchStart"
        @mouseup="dragEnd"
        @touchend="touchEnd"
        @mouseout="dragEnd"
        @touchcancel="touchEnd"
        @mousemove="mouseDraw"
        @touchmove="touchDraw"
      ></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, SetupContext, toRefs } from "vue";

type Position = {
  x: number;
  y: number;
};

export default defineComponent({
  props: {
    drawable: {
      type: Boolean,
      default: true,
    },
    name: {
      type: String,
      default: "",
    },
  },
  setup(props, { emit }: SetupContext) {
    const pointX = ref<number>(0);
    const canvasRect = ref<DOMRect | null>(null);
    const prevPosition = ref<Position>({ x: 0, y: 0 });
    const canvas = ref<HTMLCanvasElement | null>(null);
    const context = ref<CanvasRenderingContext2D | null>();
    const isDrag = ref(false);
    const canvasWidth = ref(600);
    const canvasHeight = ref(400);

    // 線を描く
    const mouseDraw = (e: MouseEvent): void => {
      if (context.value && canvasRect.value) {
        draw(e.clientX - canvasRect.value.x, e.clientY - canvasRect.value.y);
      }
    };
    const touchDraw = (e: TouchEvent): void => {
      if (context.value && canvasRect.value) {
        draw(
          e.targetTouches[0].clientX - canvasRect.value.x,
          e.targetTouches[0].clientY - canvasRect.value.y
        );
      }
    };
    const draw = (posX: number, posY: number): void => {
      if (!isDrag.value) {
        return;
      }
      if (context.value && canvasRect.value) {
        context.value.beginPath();
        context.value.moveTo(prevPosition.value.x, prevPosition.value.y);
        context.value.lineTo(posX, posY);
        context.value.stroke();
        prevPosition.value = {
          x: posX,
          y: posY,
        };
        emit("draw", { x: posX, y: posY });
      }
    };

    // お絵かき開始
    const dragStart = (e: MouseEvent): void => {
      if (context.value && canvasRect.value) {
        drawStart(e.clientX - canvasRect.value.x, e.clientY - canvasRect.value.y);
      }
    };
    const touchStart = (e: TouchEvent): void => {
      if (context.value && canvasRect.value) {
        drawStart(
          e.targetTouches[0].clientX - canvasRect.value.x,
          e.targetTouches[0].clientY - canvasRect.value.y
        );
      }
    };
    const drawStart = (x: number, y: number): void => {
      if (!canvasRect.value) return;
      isDrag.value = true;
      prevPosition.value = {
        x,
        y,
      };
      emit("draw-start", { x, y });
    };

    // お絵かき終了
    const dragEnd = (): void => {
      drawEnd();
    };
    const touchEnd = (): void => {
      drawEnd();
    };
    const drawEnd = (): void => {
      isDrag.value = false;
      emit("draw-end");
    };

    const deleteAll = (): void => {
      context.value?.clearRect(0, 0, canvasWidth.value, canvasHeight.value);
      emit("delete-all");
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
      mouseDraw,
      touchDraw,
      dragStart,
      touchStart,
      drawStart,
      dragEnd,
      touchEnd,
      drawEnd,
      draw,
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
.drawable-canvas {
  border: 1px solid #aaa;
  border-radius: 5px;
}
</style>
