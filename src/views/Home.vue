<template>
  <div class="home">
    <Canvas />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Canvas from "@/components/Canvas.vue"; // @ is an alias to /src

export default defineComponent({
  name: "Home",
  components: {
    Canvas,
  },
});
</script>
