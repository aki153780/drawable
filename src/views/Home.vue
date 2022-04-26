<template>
  <div class="home">
    <div class="flex justify-center m-4">
      <div class="w-1/5">
        <label
          class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4"
          for="inline-full-name"
        >
          名前：
        </label>
      </div>
      <div class="w-3/5">
        <input
          type="text"
          id="home--name-input"
          v-model="name"
          class="shadow appearance-none border border-500 rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
        />
      </div>
      <div class="w-1/5">
        <button
          class="w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 mx-2 rounded"
          @click="registerName"
        >
          登録
        </button>
      </div>
    </div>
    <drawing-canvas
      name="aaa"
      @draw="sendDraw"
      @draw-start="sendStartDraw"
      @draw-end="sendEndDraw"
      @delete-all="sendDeleteAll"
    />
    <drawing-canvas name="displayCanvas" ref="displayCanvas" :drawable="false" class="mt-4" />
  </div>
</template>

<script lang="ts">
import { defineComponent, SetupContext, ref } from "vue";
import DrawingCanvas from "@/components/drawing-canvas.vue"; // @ is an alias to /src
import { io } from "socket.io-client";

export default defineComponent({
  name: "Home",
  components: {
    DrawingCanvas,
  },
  setup() {
    const displayCanvas = ref();
    const name = ref<string>("");

    const socket = io("http://localhost:3000");
    socket.on("connect", () => {
      console.log(socket.id);
    });
    socket.on("draw line", (payload) => {
      console.log(payload);
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      displayCanvas.value.draw(x, y);
    });
    socket.on("start draw", (payload) => {
      console.log(payload);
      const { x, y } = payload;
      if (!displayCanvas.value) return;
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

    const registerName = () => {
      socket.emit("register name", name.value);
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
      if (!displayCanvas.value) return;
      displayCanvas.value.deleteAll();
      socket.emit("delete all");
    };
    return {
      name,
      displayCanvas,
      registerName,
      sendDraw,
      sendStartDraw,
      sendEndDraw,
      sendDeleteAll,
    };
  },
});
</script>
<style scoped>
.home--name-input {
  border: 1px solid #333;
  border-radius: 3px;
  padding: 5px;
}
</style>
