<template>
  <div class="fixed top-0 bottom-0 left-0 right-0 pointer-events-none" style="z-index: -1">
    <canvas ref="el" width="400" height="400" />
  </div>
</template>

<script setup lang="ts">
import { isDark } from '~/composables'
const el = ref<HTMLCanvasElement | null>(null)
const ctx = ref<CanvasRenderingContext2D | null>(null)
const animationFrameSequence = ref(0)
const size = reactive(useWindowSize())

const initRoundPopulation = 60

const round = reactive<RoundItem[]>([])

function initCanvas(canvas: HTMLCanvasElement, width = 400, height = 400) {
  ctx.value = canvas.getContext('2d')
  const dpr = window.devicePixelRatio || 1
  // @ts-expect-error vendor
  const bsr = ctx.value.webkitBackingStorePixelRatio || ctx.value.mozBackingStorePixelRatio || ctx.value.msBackingStorePixelRatio || ctx.value.oBackingStorePixelRatio || ctx.value.backingStorePixelRatio || 1
  // 操作 dpi 防止移动端变形
  const dpi = dpr / bsr
  canvas.style.width = `${width}px`
  canvas.style.height = `${height}px`
  canvas.width = dpi * width
  canvas.height = dpi * height
  ctx.value?.scale(dpi, dpi)
}

class RoundItem {
  constructor(index: number, x: number, y: number) {
    this.index = index
    this.x = x
    this.y = y
    this.r = Math.random() * 2 + 1
    const alpha = (Math.floor(Math.random() * 10) + 1) / 10 / 2
    this.color = isDark.value ? `rgba(255, 255, 255, ${alpha})` : `rgba(188, 188, 188, ${alpha})`
  }

  draw() {
    ctx.value.fillStyle = this.color
    ctx.value.shadowBlur = this.r * 2
    ctx.value.beginPath()
    ctx.value.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false)
    ctx.value.closePath()
    ctx.value.fill()
  }

  move() {
    this.y -= 0.15
    if (this.y <= -5)
      this.y = size.height + 10

    this.draw()
  }
}

function animate() {
  ctx.value.clearRect(0, 0, size.width, size.height)
  for (const i in round)
    round[i].move()
  animationFrameSequence.value = requestAnimationFrame(animate)
}

function initStars() {
  round.length = 0
  for (let i = 0; i < initRoundPopulation; i++) {
    round[i] = new RoundItem(i, Math.random() * size.width, Math.random() * size.height)
    round[i].draw()
  }
}

function main() {
  const canvas = el.value
  initCanvas(canvas, size.width, size.height)
  initStars()
  animate()
}

onMounted(() => {
  main()
})

function clear() {
  // 清除动画 否则切换主题之后动画的速度会越来越快
  cancelAnimationFrame(animationFrameSequence.value)
  ctx.value?.clearRect(0, 0, size.width, size.height)
}

watch(isDark, (val) => {
  nextTick(() => {
    clear()
    main()
  })
})
</script>
