<template lang="pug">
.mx-4
</template>

<script setup lang="ts">
class ComplexArray {
  re: Float32Array;
  im: Float32Array;
  constructor(length: number) {
    this.re = new Float32Array(length);
    this.im = new Float32Array(length);
  }
}
</script>

<style scoped lang="scss">
.graph {
  height: 33vh;
  border-bottom: 1px solid black;
}
</style>
