<template>
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
</template>

<script lang="ts">
import { defineComponent, ref, SetupContext } from "vue";

export default defineComponent({
  setup(props, { emit }: SetupContext) {
    const name = ref<string>("");

    const registerName = () => {
      emit("register-name", name.value);
    };
    return {
      name,
      registerName,
    };
  },
});
</script>
