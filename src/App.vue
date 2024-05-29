<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <div class="w-full h-full flex flex-col">
    <div class="w-full h-[10%] flex justify-center space-x-3">
      <RouterLink to="/home"><span class="text-dark_green">Go to Home</span></RouterLink>
      <RouterLink to="/about"><span class="text-dark_green">Go to About</span></RouterLink>
    </div>
    <div class="w-full h-[90%]">
      <RouterView />
    </div>
  </div>
</template>
