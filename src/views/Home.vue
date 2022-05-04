<template>
  <div class="home">
    <register-name @registerName="registerName" />
    <drawing-canvas
      v-if="isDrawer && isNameRegistered"
      :width="width"
      :height="height"
      @draw="sendDraw"
      @draw-start="sendStartDraw"
      @draw-end="sendEndDraw"
      @delete-all="sendDeleteAll"
    />
    <drawing-canvas
      v-show="!isDrawer && isNameRegistered"
      name="displayCanvas"
      :width="width"
      :height="height"
      ref="displayCanvas"
      :drawable="false"
      class="mt-4"
    />
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
    const width = 600;
    const height = 400;

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
      if (!displayCanvas.value) return;
      displayCanvas.value.deleteAll();
      socket.emit("delete all");
    };
    return {
      width,
      height,
      isDrawer,
      isNameRegistered,
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
