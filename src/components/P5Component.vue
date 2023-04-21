<template>
  <div ref="canvas" />
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import p5 from 'p5'

export default {
  name: 'P5Component',
  setup() {
    const canvas = ref<HTMLCanvasElement | undefined>(undefined)

    onMounted(() => {
      new p5((p: p5) => {
        p.setup = () => {
          p.createCanvas(400, 400)
        }

        p.draw = () => {
          p.background(0)
          p.fill(255)
          p.rect(100, 100, 200, 200)
        }
      }, canvas.value)
    });
    return { canvas }
  }
}
</script>
