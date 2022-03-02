<template>
  <div class="home">
    <Canvas
      name="aaa"
      @draw="sendDraw"
      @draw-start="sendStartDraw"
      @draw-end="sendEndDraw"
      @delete-all="sendDeleteAll"
    />
    <Canvas
      name="displayCanvas"
      ref="displayCanvas"
      :drawable="false"
      class="mt-4"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, SetupContext, ref } from "vue";
import Canvas from "@/components/Canvas.vue"; // @ is an alias to /src

export default defineComponent({
  name: "Home",
  components: {
    Canvas,
  },
  setup() {
    const displayCanvas = ref();

    const sendDraw = (payload: { x: number; y: number }): void => {
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      displayCanvas.value.draw(x, y);
    };
    const sendStartDraw = (payload: { x: number; y: number }): void => {
      const { x, y } = payload;
      if (!displayCanvas.value) return;
      displayCanvas.value.drawStart(x, y);
    };
    const sendEndDraw = (): void => {
      if (!displayCanvas.value) return;
      displayCanvas.value.drawEnd();
    };
    const sendDeleteAll = (): void => {
      if (!displayCanvas.value) return;
      displayCanvas.value.deleteAll();
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
