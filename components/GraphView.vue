<template lang="pug">
canvas.w-100(ref="canvas")
</template>

<script setup lang="ts">
const canvas = ref<HTMLCanvasElement>();
const props = defineProps<{ data: number[][]; logScale?: boolean }>();
const draw = (context: CanvasRenderingContext2D) => {
  const width = (canvas.value.width = canvas.value.offsetWidth);
  const height = (canvas.value.height = canvas.value.offsetHeight);
  let min = props.data[0]?.[0] || 0;
  let max = props.data[0]?.[0] || 0;
  for (const data of props.data) {
    for (const current of data) {
      min = Math.min(current, min);
      max = Math.max(current, max);
    }
  }
  let index = 0;
  for (const data of props.data) {
    context.beginPath();
    context.moveTo(0, (1 - (data[0] - min) / (max - min)) * height);
    for (var i = 0; i < data.length; i++) {
      const current = data[i];
      context.lineTo(
        !props.logScale
          ? (i / (data.length - 1)) * width
          : (-Math.pow(1 - i / (data.length - 1), 10) + 1) * width,
        (1 - (current - min) / (max - min)) * height
      );
    }
    context.strokeStyle = `rgb(${index === 0 ? 255 : 0},${
      index === 1 ? 255 : 0
    },0)`;
    context.stroke();
    index++;
  }
  context.font = "24px serif";
  context.fillText(min.toString(), 0, height);
  context.fillText(max.toString(), 0, 24);
};
onMounted(() => {
  watch(
    props,
    () => {
      const context = canvas.value.getContext("2d");
      draw(context);
    },
    { deep: true, immediate: true }
  );
});
</script>
