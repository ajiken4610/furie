<template lang="pug">
.mx-4
  GraphView.graph(:data="graphData")
  GraphView.graph(:data="resultGraphData")
  GraphView.graph(:data="reversedGraphData")
</template>

<script setup lang="ts">
class ComplexArray {
  re: Float32Array;
  im: Float32Array;
  length: number;
  constructor(length: number) {
    this.length = length;
    this.re = new Float32Array(length);
    this.im = new Float32Array(length);
  }
  invert(result: ComplexArray) {
    for (var i = 0; i < this.length; i++) {
      result.re[i] = this.re[i];
      result.im[i] = -this.im[i];
    }
  }
}

const createComplexArray = (length: number, func: (val: number) => number) => {
  const ret = new ComplexArray(length);
  for (var i = 0; i < length; i++) {
    ret.re[i] = func(i / length);
  }
  return ret;
};

const deg = 8;
const ndeg = 4 ** deg;
const data = createComplexArray(ndeg, (t) => {
  const cos = [12, 5, 16];
  const sin = [1, 8];
  return (
    Math.random() -
    0.5 +
    cos.reduce((sum, val) => sum + Math.cos(t * 2 * Math.PI * val), 0) +
    sin.reduce((sum, val) => sum + Math.sin(t * 2 * Math.PI * val), 0)
  );
});
const graphData = (() => {
  const ret: number[] = [];
  for (const current of data.re) {
    ret.push(current);
  }
  return [ret];
})();
const result = createComplexArray(ndeg, () => 0);

const transform = (data: ComplexArray, result: ComplexArray) => {
  for (var i = 1; i < deg + 1; i++) {
    const pdeg = 4 ** (deg - i);
    for (var j0 = 0; j0 < 4 ** (i - 1); j0++) {
      for (var j1 = 0; j1 < pdeg; j1++) {
        const j = j1 + j0 * pdeg * 4;
        // バタフライ演算(Q=4)
        const w1 =
          data.re[j] +
          data.re[j + pdeg] +
          data.re[j + 2 * pdeg] +
          data.re[j + 3 * pdeg];
        const w2 =
          data.im[j] +
          data.im[j + pdeg] +
          data.im[j + 2 * pdeg] +
          data.im[j + 3 * pdeg];
        const w3 =
          data.re[j] +
          data.im[j + pdeg] -
          data.re[j + 2 * pdeg] -
          data.im[j + 3 * pdeg];
        const w4 =
          data.im[j] -
          data.re[j + pdeg] -
          data.im[j + 2 * pdeg] +
          data.re[j + 3 * pdeg];
        const w5 =
          data.re[j] -
          data.re[j + pdeg] +
          data.re[j + 2 * pdeg] -
          data.re[j + 3 * pdeg];
        const w6 =
          data.im[j] -
          data.im[j + pdeg] +
          data.im[j + 2 * pdeg] -
          data.im[j + 3 * pdeg];
        const w7 =
          data.re[j] -
          data.im[j + pdeg] -
          data.re[j + 2 * pdeg] +
          data.im[j + 3 * pdeg];
        const w8 =
          data.im[j] +
          data.re[j + pdeg] -
          data.im[j + 2 * pdeg] -
          data.re[j + 3 * pdeg];
        data.re[j] = w1;
        data.im[j] = w2;
        data.re[j + pdeg] = w3;
        data.im[j + pdeg] = w4;
        data.re[j + 2 * pdeg] = w5;
        data.im[j + 2 * pdeg] = w6;
        data.re[j + 3 * pdeg] = w7;
        data.im[j + 3 * pdeg] = w8;
        // 回転因子
        for (var k = 0; k < 4; k++) {
          const w1 = Math.cos((2 * Math.PI * j * k) / pdeg / 4);
          const w2 = -Math.sin((2 * Math.PI * j * k) / pdeg / 4);
          const w3 = data.re[j + k * pdeg] * w1 - data.im[j + k * pdeg] * w2;
          const w4 = data.re[j + k * pdeg] * w2 + data.im[j + k * pdeg] * w1;
          data.re[j + k * pdeg] = w3;
          data.im[j + k * pdeg] = w4;
        }
      }
    }
  }
  // ビット反転
  for (var i = 0; i < ndeg; i++) {
    let k = i;
    let k1 = 0;
    for (var j = 1; j < deg + 1; j++) {
      k1 = k1 + (k - Math.floor(k / 4) * 4) * 4 ** (deg - j);
      k = Math.floor(k / 4);
    }
    result.re[i] = data.re[k1];
    result.im[i] = data.im[k1];
  }
};
const startTime = Date.now();
transform(data, result);
result.re = result.re.slice(0, ndeg / 2 + 1);
result.im = result.im.slice(0, ndeg / 2 + 1);
const finishTime = Date.now();
console.log(finishTime - startTime);

const resultGraphData = (() => {
  const ret: [number[], number[]] = [[], []];
  for (const current of result.re) {
    ret[0].push(current);
  }
  for (const current of result.im) {
    ret[1].push(current);
  }
  return ret;
})();

const reversedGraphData = computed(() => {
  const inverted = new ComplexArray(ndeg);
  result.invert(inverted);
  for (var i = 0; i < ndeg / 2; i++) {
    inverted.re[ndeg - i] = inverted.re[i];
    inverted.im[ndeg - i] = -inverted.im[i];
  }
  const reversed = new ComplexArray(ndeg);

  transform(inverted, reversed);
  reversed.invert(reversed);
  const ret: number[] = [];
  for (var i = 0; i < ndeg; i++) {
    ret.push((reversed.re[i] + reversed.im[i]) / ndeg);
  }
  return [ret];
});
</script>
<style scoped lang="scss">
.graph {
  height: 33vh;
  border-bottom: 1px solid black;
}
</style>
