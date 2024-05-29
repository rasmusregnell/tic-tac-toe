<script setup lang="ts">
import { RouterLink } from "vue-router";
</script>

<template>
  <div class="w-full h-full flex justify-center items-center">
    <div
      class="border-2 border-dark_green rounded-full w-[200px] h-[50px] flex justify-center items-center"
    >
      <RouterLink to="/play" class="text-dark_green p-3 text-2xl"
        >Press to play</RouterLink
      >
    </div>
  </div>
</template>
