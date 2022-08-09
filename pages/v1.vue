<template lang="pug">
.mx-4
  GraphView.graph(:data="graphData")
  GraphView.graph(:data="resultGraphData")
</template>

<script setup lang="ts">
class Complex {
  re: number;
  im: number;
  constructor(re?: number, im?: number) {
    this.re = re || 0;
    this.im = im || 0;
  }
  add(b: Complex) {
    return new Complex(this.re + b.re, this.im + b.im);
  }
  mul(b: Complex) {
    return new Complex(
      this.re * b.re - this.im * b.im,
      this.re * b.im + this.im * b.re
    );
  }
  toString() {
    return `(${Math.round(this.re * 10) / 10},${
      Math.round(this.im * 10) / 10
    })`;
  }
  invert() {
    return new Complex(this.re, -this.im);
  }
}
class Mat {
  array: Complex[];
  size: number;
  constructor(size: number) {
    this.size = size;
    this.array = new Array(size * size);
  }
  setAt(x: number, y: number, value: Complex) {
    this.array[x + y * this.size] = value;
  }
  getAt(x: number, y: number) {
    return this.array[x + y * this.size];
  }
  mul(b: Complex[]) {
    const ret: Complex[] = Array(this.size);
    for (var i = 0; i < this.size; i++) {
      let sum = new Complex();
      for (var j = 0; j < this.size; j++) {
        sum = sum.add(this.getAt(i, j).mul(b[j]));
      }
      ret[i] = sum;
    }
    return ret;
  }
  toString() {
    let ret = "";
    for (var i = 0; i < this.size; i++) {
      for (var j = 0; j < this.size; j++) {
        ret += this.getAt(i, j) + " ";
      }
      ret += "\n";
    }
    return ret;
  }
}
const createComplexVec = (length: number, func: (val: number) => number) => {
  const ret: Complex[] = Array(length);
  for (var i = 0; i < length; i++) {
    ret[i] = new Complex(func(i / length));
  }
  return ret;
};

const createMat = (size: number) => {
  const ret = new Mat(size);
  for (var i = 0; i < size; i++) {
    for (var j = 0; j < size; j++) {
      ret.setAt(
        i,
        j,
        new Complex(
          Math.cos((2 * Math.PI * i * j) / size),
          -Math.sin((2 * Math.PI * i * j) / size)
        )
      );
    }
  }
  return ret;
};

const size = 4 ** 8;
const data = createComplexVec(size, (t) => {
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
const startTime = Date.now();
const result = mat.mul(data).slice(0, size / 2 + 1);
const finishTime = Date.now();
console.log(finishTime - startTime);

// console.log(result.join("\n"));

const graphData = computed(() => {
  const ret: number[] = [];
  for (const current of data) {
    ret.push(current.re);
  }
  return [ret];
});

const resultGraphData = computed(() => {
  const ret: [number[], number[]] = [[], []];
  for (const current of result) {
    ret[0].push(current.re / size);
    ret[1].push(current.im / size);
  }
  return ret;
});
</script>

<style scoped lang="scss">
.graph {
  height: 33vh;
  border-bottom: 1px solid black;
}
</style>
