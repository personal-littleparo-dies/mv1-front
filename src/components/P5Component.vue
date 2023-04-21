<template>
  <div ref="canvas" />
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import * as p5 from 'p5'

export default {
  name: 'P5Component',
  setup() {
    const canvas = ref<HTMLCanvasElement | null>(null)
    onMounted(() => {
      new p5((sketch) => {
        sketch.setup = () => {
          sketch.createCanvas(400, 400)
        }

        sketch.draw = () => {
          sketch.background(0)
          sketch.fill(255)
          sketch.rect(100, 100, 200, 200)
        }
      }, canvas.value)
    })
    return { canvas }
  }
}
</script>
