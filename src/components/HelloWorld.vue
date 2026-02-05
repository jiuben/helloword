<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

defineProps<{ msg: string }>()

const scroller = ref<HTMLElement | null>(null)
const containerWidth = ref(0)
const scrollTop = ref(0)
const viewportHeight = ref(600)

const totalItems = 1500
const items = Array.from({ length: totalItems }, (_, i) => ({ id: i, seed: i }))

// increase card size per user request
const itemWidth = 220
const itemHeight = 200
const gap = 16
const rowHeight = itemHeight + gap

const columns = computed(() => Math.max(1, Math.floor(containerWidth.value / (itemWidth + gap))))
const rowCount = computed(() => Math.ceil(items.length / columns.value))
const totalHeight = computed(() => rowCount.value * rowHeight)

const bufferRows = 3

function onScroll(e: Event) {
  const t = e.target as HTMLElement
  scrollTop.value = t.scrollTop
}

let ro: ResizeObserver | null = null

onMounted(() => {
  if (scroller.value) {
    viewportHeight.value = scroller.value.clientHeight
    containerWidth.value = scroller.value.clientWidth
    ro = new ResizeObserver(() => {
      if (!scroller.value) return
      viewportHeight.value = scroller.value.clientHeight
      containerWidth.value = scroller.value.clientWidth
    })
    ro.observe(scroller.value)
    scroller.value.addEventListener('scroll', onScroll, { passive: true })
  }
})

onUnmounted(() => {
  if (ro && scroller.value) ro.unobserve(scroller.value)
  if (scroller.value) scroller.value.removeEventListener('scroll', onScroll)
})

const visibleRange = computed(() => {
  const startRow = Math.max(0, Math.floor(scrollTop.value / rowHeight) - bufferRows)
  const endRow = Math.min(rowCount.value - 1, Math.ceil((scrollTop.value + viewportHeight.value) / rowHeight) + bufferRows)
  const startIndex = startRow * columns.value
  const endIndex = Math.min(items.length, (endRow + 1) * columns.value)
  return { startIndex, endIndex }
})

const visibleIndices = computed(() => {
  const out: number[] = []
  for (let i = visibleRange.value.startIndex; i < visibleRange.value.endIndex; i++) out.push(i)
  return out
})

function imageUrl(i: number) {
  // picsum provides open-source placeholder images; request higher-res versions
  const w = Math.max(600, itemWidth * 2)
  const h = Math.max(400, itemHeight * 2)
  return `https://picsum.photos/seed/${i}/${w}/${h}`
}

</script>

<template>
  <section class="page">
    <header class="hero">
      <h1 class="title">{{ msg || '漂亮的画廊' }}</h1>
      <p class="subtitle">响应式流布局 · 动画 · 虚拟滚动（高性能）</p>
    </header>

    <div ref="scroller" class="scroller">
      <div class="inner" :style="{ height: totalHeight + 'px' }">
        <div
          v-for="i in visibleIndices"
          :key="items[i].id"
          class="card"
          :style="(() => {
            const row = Math.floor(i / columns)
            const col = i % columns
            const top = row * rowHeight
            const left = col * (itemWidth + gap)
            return {
              transform: `translate(${left}px, ${top}px)`,
              width: itemWidth + 'px',
              height: itemHeight + 'px'
            }
          })()"
        >
          <div class="thumb">
            <img :src="imageUrl(i)" :alt="`img-${i}`" loading="lazy" />
          </div>
          <div class="meta">
            <div class="tag">#{{ i }}</div>
            <div class="fade">来自 picsum.photos</div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
:root {
  --bg: #0f1724;
  --card: #0b1220;
  --accent: linear-gradient(90deg, #7c3aed 0%, #06b6d4 100%);
}
.page {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
  color: #e6eef8;
  background: radial-gradient(1200px 400px at 10% 10%, rgba(124,58,237,0.15), transparent), #071023;
  min-height: 100vh;
  padding: 28px;
}
.hero {
  margin-bottom: 18px;
}
.title {
  font-size: 28px;
  margin: 0 0 6px 0;
  background: var(--accent);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
.subtitle { color: #9fb0c8; margin: 0; font-size: 13px }

.scroller {
  width: 50vw;
  height: calc(100vh - 140px);
  overflow: auto;
  position: relative;
  border-radius: 12px;
  padding: 16px;
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  box-shadow: 0 6px 30px rgba(2,6,23,0.6) inset;
}
.inner { position: relative; width: 100%; }

.card {
  position: absolute;
  border-radius: 10px;
  overflow: hidden;
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  box-shadow: 0 6px 18px rgba(2,6,23,0.6);
  transition: transform 320ms cubic-bezier(.2,.9,.2,1), box-shadow 200ms;
  will-change: transform;
  display: flex;
  flex-direction: column;
}
.card:hover { transform: translateY(-6px) scale(1.02); box-shadow: 0 12px 30px rgba(2,6,23,0.7); }
.thumb { flex: 1 1 auto; background: #0a1220; display: flex; align-items: center; justify-content: center; }
.thumb img { width: 100%; height: 100%; object-fit: cover; display:block; opacity:0; animation: fadeIn 500ms forwards; }

.meta { padding: 8px 10px; display:flex; justify-content:space-between; align-items:center; font-size:12px; color:#bcd3e6 }
.tag { background: rgba(255,255,255,0.03); padding:4px 8px; border-radius:999px }
.fade { opacity:0.8 }

@keyframes fadeIn { to { opacity: 1 } }

/* small screens tweak */
@media (max-width: 520px) {
  .title { font-size: 20px }
}
</style>
