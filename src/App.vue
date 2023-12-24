<styles lang="scss" scoped>
.main {
  position: absolute;
  bottom: 25%;
  left: 50%;
  transform: translateX(-50%);
  width: 800px;
}
</styles>

<template>
  <v-app>
    <v-main class="main">
      <Balloon />
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import Balloon from "@/components/Balloon.vue";
</script>
