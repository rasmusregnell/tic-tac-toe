<script setup lang="ts">
import { RouterLink } from "vue-router";
</script>

<template>
  <div class="w-full h-full flex flex-col items-center">
    <span class="mb-2 text-[100px] text-dark_green mt-[200px]"
      >Tic Tac Toe</span
    >
    <div
      class="border-2 border-dark_green rounded-full w-[200px] h-[50px] flex justify-center items-center"
    >
      <RouterLink to="/play" class="text-dark_green p-3 text-2xl"
        >Press to play</RouterLink
      >
    </div>
  </div>
</template>
