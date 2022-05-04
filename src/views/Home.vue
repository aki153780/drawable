<template>
  <div class="home">
    <register-name @registerName="registerName" />
    <template v-if="isDrawer && isNameRegistered">
      <div class="m-4">
        <button
          class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 mr-2 rounded disabled:opacity-25"
          @click="sendDeleteAll"
          :disabled="drawable"
        >
          すべて削除
        </button>
      </div>
      <div class="w-full flex justify-center">
        <drawing-canvas
          ref="drawingCanvas"
          :width="width"
          :height="height"
          :drawable="drawable"
          class="drawable-canvas"
          @draw="sendDraw"
          @draw-start="sendStartDraw"
          @draw-end="sendEndDraw"
        />
      </div>
    </template>
    <div v-show="!isDrawer && isNameRegistered">
      <div class="m-4">
        <button
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 mr-2 rounded"
          @click="proposeAnswer"
        >
          回答する
        </button>
      </div>
      <div class="w-full flex justify-center">
        <drawing-canvas
          name="displayCanvas"
          :width="width"
          :height="height"
          ref="displayCanvas"
          :drawable="false"
          class="mt-4 drawable-canvas"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, SetupContext, ref, onBeforeUnmount } from "vue";
import DrawingCanvas from "@/components/drawing-canvas.vue";
import RegisterName from "@/components/register-name.vue";
import { io } from "socket.io-client";

export default defineComponent({
  name: "Home",
  components: {
    RegisterName,
    DrawingCanvas,
  },
  setup() {
    const displayCanvas = ref();
    const drawingCanvas = ref();
    const width = 600;
    const height = 400;
    const drawable = ref(true);

    const socket = io("http://localhost:3000");
    socket.on("connect", () => {
      console.log(socket.id);
    });

    socket.on("draw line", (payload) => {
      console.log("受信：", payload, displayCanvas.value);
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      console.log("draw line");
      displayCanvas.value.draw(x, y);
    });
    socket.on("start draw", (payload) => {
      console.log(payload);
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      console.log("start draw");
      displayCanvas.value.drawStart(x, y);
    });
    socket.on("end draw", () => {
      if (!displayCanvas.value) return;
      displayCanvas.value.drawEnd();
    });
    socket.on("delete all", () => {
      if (!displayCanvas.value) return;
      displayCanvas.value.deleteAll();
    });
    socket.on("stop drawing", () => {
      console.log("stop drawing");
      if (drawingCanvas.value) drawable.value = false;
    });
    socket.on("failed answering", () => {
      if (drawingCanvas.value) drawable.value = true;
    });

    const isDrawer = ref(true);
    socket.on("drawer changed", (drawer: string) => {
      console.log(socket.id, drawer);
      isDrawer.value = socket.id === drawer;
      console.log(isDrawer.value);
    });

    const isNameRegistered = ref(false);
    const registerName = (name: string) => {
      socket.emit("register name", name);
      isNameRegistered.value = true;
    };

    const sendDraw = (payload: { x: number; y: number }): void => {
      socket.emit("draw line", payload);
    };
    const sendStartDraw = (payload: { x: number; y: number }): void => {
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      displayCanvas.value.drawStart(x, y);
      socket.emit("start draw", payload);
    };
    const sendEndDraw = (): void => {
      if (!displayCanvas.value) return;
      displayCanvas.value.drawEnd();
      socket.emit("end draw");
    };
    const sendDeleteAll = (): void => {
      if (displayCanvas.value) displayCanvas.value.deleteAll();
      if (drawingCanvas.value) drawingCanvas.value.deleteAll();
      socket.emit("delete all");
    };
    const proposeAnswer = (): void => {
      socket.emit("propose answer");
    };
    return {
      width,
      height,
      drawable,
      isDrawer,
      isNameRegistered,
      displayCanvas,
      drawingCanvas,
      registerName,
      sendDraw,
      sendStartDraw,
      sendEndDraw,
      sendDeleteAll,
      proposeAnswer,
    };
  },
});
</script>
<style>
.drawable-canvas {
  border: 1px solid #aaa;
  border-radius: 5px;
}
</style>
