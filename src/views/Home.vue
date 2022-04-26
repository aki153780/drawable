<template>
  <div class="home">
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
      displayCanvas,
      sendDraw,
      sendStartDraw,
      sendEndDraw,
      sendDeleteAll,
    };
  },
});
</script>
