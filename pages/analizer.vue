<template lang="pug">
div
  button(v-if="!started" @click="start") Click
  div(v-else-if="freq&&domain")
    GraphView.graph(:data="freq" :logScale="true")
    GraphView.graph(:data="domain" :offset="domainOffset")
</template>

<script setup lang="ts">
const started = ref(false);
const freq = ref();
const domain = ref();
const domainOffset = ref(0);
const start = async () => {
  started.value = true;
  const audioCtx = new AudioContext();
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  const input = audioCtx.createMediaStreamSource(stream);
  const analyzer = audioCtx.createAnalyser();
  input.connect(analyzer);
  const timeDomainArray = new Float32Array(analyzer.fftSize);
  const frequencyArray = new Float32Array(analyzer.frequencyBinCount);

  const render = () => {
    analyzer.getFloatFrequencyData(frequencyArray);
    analyzer.getFloatTimeDomainData(timeDomainArray);
    freq.value = [frequencyArray];
    domain.value = [timeDomainArray];
    requestAnimationFrame(render);
  };
  render();
};
</script>
<style scoped lang="scss">
.graph {
  height: 33vh;
  border-bottom: 1px solid black;
}
</style>
