<template lang="pug">
.mx-4
  GraphView.graph(:data="graphData")
  GraphView.graph(:data="resultGraphData")
  GraphView.graph(:data="reversedGraphData")
  GraphView.graph(:data="[diff]")
  GraphView.graph(:data="cosTrans")
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
  pow(exp: number, result: ComplexArray) {
    for (var i = 0; i < this.length; i++) {
      const cRi = this.re[i];
      const cIm = this.im[i];
      const length = Math.sqrt(cRi * cRi + cIm * cIm) ** exp;
      const arg = Math.atan2(cIm, cRi) * exp;
      result.re[i] = length * Math.cos(arg);
      result.im[i] = length * Math.sin(arg);
    }
  }
}
class Mat {
  data: ComplexArray[];
  size: number;
  constructor(size: number) {
    this.size = size;
    this.data = Array(size);
    for (var i = 0; i < size; i++) {
      this.data[i] = new ComplexArray(size);
    }
  }
  mul(b: ComplexArray, result: ComplexArray) {
    for (var i = 0; i < this.size; i++) {
      let re = 0;
      let im = 0;
      const current = this.data[i];
      for (var j = 0; j < this.size; j++) {
        const ar = current.re[j],
          br = b.re[j],
          ai = current.im[j],
          bi = b.im[j];
        re += ar * br - ai * bi;
        im += ar * bi + ai * br;
      }
      result.re[i] = re;
      result.im[i] = im;
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
const createMat = (size: number) => {
  const ret = new Mat(size);
  for (var i = 0; i < size; i++) {
    for (var j = 0; j < size; j++) {
      ret.data[i].re[j] = Math.cos((2 * Math.PI * i * j) / size);
      ret.data[i].im[j] = -Math.sin((2 * Math.PI * i * j) / size);
    }
  }
  return ret;
};

const size = 4 ** 6;
const data = createComplexArray(size, (t) => {
  const cos = [12, 5, 16];
  const sin = [1, 8];
  return (
    Math.random() -
    0.5 +
    cos.reduce((sum, val) => sum + Math.cos(t * 2 * Math.PI * val), 0) +
    sin.reduce((sum, val) => sum + Math.sin(t * 2 * Math.PI * val), 0)
  );
});
const mat = createMat(size);
const result = new ComplexArray(size);
const startTime = Date.now();
mat.mul(data, result);
result.re = result.re.slice(0, size / 2 + 1);
result.im = result.im.slice(0, size / 2 + 1);
const finishTime = Date.now();
console.log(finishTime - startTime);

const graphData = computed(() => {
  const ret: number[] = [];
  for (const current of data.re) {
    ret.push(current);
  }
  return [ret];
});

const resultGraphData = computed(() => {
  const ret: [number[], number[]] = [[], []];
  for (const current of result.re) {
    ret[0].push(current);
  }
  for (const current of result.im) {
    ret[1].push(current);
  }
  return ret;
});

const diff = ref<number[]>([]);

const reversedGraphData = computed(() => {
  const inverted = new ComplexArray(size);
  result.invert(inverted);
  for (var i = 0; i < size / 2; i++) {
    inverted.re[size - i] = inverted.re[i];
    inverted.im[size - i] = -inverted.im[i];
  }
  const reversed = new ComplexArray(size);

  mat.mul(inverted, reversed);
  reversed.invert(reversed);
  const ret: number[] = [];
  for (var i = 0; i < size; i++) {
    ret.push((reversed.re[i] + reversed.im[i]) / size);
  }

  diff.value = [];
  for (var i = 0; i < size; i++) {
    diff.value.push((reversed.re[i] + reversed.im[i]) / size - data.re[i]);
  }
  return [ret];
});

const cosTrans = computed(() => {
  const fac: number[] = Array(size / 2 + 1);
  const arg: number[] = Array(size / 2 + 1);
  for (var i = 0; i < size / 2 + 1; i++) {
    fac[i] =
      Math.sqrt(result.re[i] * result.re[i] + result.im[i] * result.im[i]) /
      size;
    arg[i] = Math.atan2(result.im[i], result.re[i]);
  }
  fac[0] /= 2;
  const ret: number[] = Array(size);
  ret.fill(0);
  for (var i = 0; i < size / 2 + 1; i++) {
    for (var j = 0; j < size; j++) {
      ret[j] += 2 * fac[i] * Math.cos((2 * Math.PI * i * j) / size + arg[i]);
    }
  }
  // let toString: string[] = [];
  // for (var i = 0; i < size / 2 + 1; i++) {
  //   toString.push(
  //     `${(2 * fac[i]).toFixed(5)}cos(${(2 * i).toFixed(5)}pix-${arg[i].toFixed(
  //       5
  //     )})`
  //   );
  // }
  // console.log(toString.join("+"));
  return [ret];
});
</script>

<style scoped lang="scss">
.graph {
  height: 33vh;
  border-bottom: 1px solid black;
}
</style>
