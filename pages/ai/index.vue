<template>
  <div class="ai-page anim-in">
    <div class="ai-chat-panel">
      <canvas ref="canvas" class="neural-bg"></canvas>

      <!-- Header -->
      <div class="chat-head">
        <div class="chat-head-l">
          <div class="ai-logo-wrap">
            <svg width="22" height="22" viewBox="0 0 40 40" fill="none">
              <circle cx="20" cy="20" r="7" stroke="url(#nh1)" stroke-width="2.5" fill="none"/>
              <circle cx="8" cy="8" r="3" fill="url(#nh2)" opacity="0.8"/>
              <circle cx="32" cy="8" r="3" fill="url(#nh2)" opacity="0.8"/>
              <circle cx="8" cy="32" r="3" fill="url(#nh2)" opacity="0.8"/>
              <circle cx="32" cy="32" r="3" fill="url(#nh2)" opacity="0.8"/>
              <line x1="8" y1="8" x2="14" y2="14" stroke="url(#nh1)" stroke-width="1.5" opacity="0.6"/>
              <line x1="32" y1="8" x2="26" y2="14" stroke="url(#nh1)" stroke-width="1.5" opacity="0.6"/>
              <line x1="8" y1="32" x2="14" y2="26" stroke="url(#nh1)" stroke-width="1.5" opacity="0.6"/>
              <line x1="32" y1="32" x2="26" y2="26" stroke="url(#nh1)" stroke-width="1.5" opacity="0.6"/>
              <defs>
                <linearGradient id="nh1" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse">
                  <stop stop-color="#4dd6e8"/><stop offset="1" stop-color="#06b6d4"/>
                </linearGradient>
                <linearGradient id="nh2" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse">
                  <stop stop-color="#99e6f0"/><stop offset="1" stop-color="#67e8f9"/>
                </linearGradient>
              </defs>
            </svg>
          </div>
          <div>
            <div class="chat-title">AI <span style="color:var(--teal)">Ассистент</span></div>
            <div class="chat-sub">Спросите что угодно</div>
          </div>
        </div>
        <div class="online-pill">● Онлайн</div>
      </div>

      <!-- Messages area -->
      <div ref="area" class="chat-area">
        <div v-if="!msgs.length" class="chat-empty-state">
          <div class="ai-empty-logo">
            <svg width="40" height="40" viewBox="0 0 40 40" fill="none">
              <circle cx="20" cy="20" r="8" stroke="url(#ee1)" stroke-width="2" fill="none"/>
              <circle cx="7" cy="7" r="3.5" fill="url(#ee2)" opacity="0.9"/>
              <circle cx="33" cy="7" r="3.5" fill="url(#ee2)" opacity="0.9"/>
              <circle cx="7" cy="33" r="3.5" fill="url(#ee2)" opacity="0.9"/>
              <circle cx="33" cy="33" r="3.5" fill="url(#ee2)" opacity="0.9"/>
              <circle cx="20" cy="4" r="2.5" fill="url(#ee2)" opacity="0.7"/>
              <circle cx="36" cy="20" r="2.5" fill="url(#ee2)" opacity="0.7"/>
              <circle cx="20" cy="36" r="2.5" fill="url(#ee2)" opacity="0.7"/>
              <circle cx="4" cy="20" r="2.5" fill="url(#ee2)" opacity="0.7"/>
              <line x1="7" y1="7" x2="13" y2="13" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="33" y1="7" x2="27" y2="13" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="7" y1="33" x2="13" y2="27" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="33" y1="33" x2="27" y2="27" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="20" y1="4" x2="20" y2="12" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="36" y1="20" x2="28" y2="20" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="20" y1="36" x2="20" y2="28" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <line x1="4" y1="20" x2="12" y2="20" stroke="url(#ee1)" stroke-width="1.2" opacity="0.5"/>
              <defs>
                <linearGradient id="ee1" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse">
                  <stop stop-color="#4dd6e8"/><stop offset="1" stop-color="#06b6d4"/>
                </linearGradient>
                <linearGradient id="ee2" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse">
                  <stop stop-color="#99e6f0"/><stop offset="1" stop-color="#67e8f9"/>
                </linearGradient>
              </defs>
            </svg>
          </div>
          <p class="empty-hint">Ваш личный ИИ</p>
          <div class="empty-chips">
            <button v-for="t in tips" :key="t" class="e-chip" @click="quick(t)">{{t}}</button>
          </div>
        </div>

        <div v-else class="chat-msgs">
          <div v-for="m in msgs" :key="m.id" :class="['chat-msg', m.role]">
            <div v-if="m.role==='assistant'" class="msg-avatar">
              <div class="ai-av-icon">
                <svg width="13" height="13" viewBox="0 0 40 40" fill="none">
                  <circle cx="20" cy="20" r="8" stroke="url(#am1)" stroke-width="3" fill="none"/>
                  <circle cx="8" cy="8" r="3" fill="url(#am1)"/>
                  <circle cx="32" cy="8" r="3" fill="url(#am1)"/>
                  <circle cx="8" cy="32" r="3" fill="url(#am1)"/>
                  <circle cx="32" cy="32" r="3" fill="url(#am1)"/>
                  <defs><linearGradient id="am1" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse"><stop stop-color="#4dd6e8"/><stop offset="1" stop-color="#06b6d4"/></linearGradient></defs>
                </svg>
              </div>
              <span class="msg-sender">AI</span>
            </div>
            <!-- Image preview if user sent image -->
            <img v-if="m.imagePreview" :src="m.imagePreview" class="msg-img-preview" alt="uploaded"/>
            <div :class="['msg-bubble', m.role]" v-html="fmt(m.text)"></div>
          </div>

          <!-- Typing indicator -->
          <div v-if="loading" class="chat-msg assistant">
            <div class="msg-avatar">
              <div class="ai-av-icon">
                <svg width="13" height="13" viewBox="0 0 40 40" fill="none">
                  <circle cx="20" cy="20" r="8" stroke="url(#at1)" stroke-width="3" fill="none"/>
                  <circle cx="8" cy="8" r="3" fill="url(#at1)"/>
                  <circle cx="32" cy="8" r="3" fill="url(#at1)"/>
                  <circle cx="8" cy="32" r="3" fill="url(#at1)"/>
                  <circle cx="32" cy="32" r="3" fill="url(#at1)"/>
                  <defs><linearGradient id="at1" x1="0" y1="0" x2="40" y2="40" gradientUnits="userSpaceOnUse"><stop stop-color="#4dd6e8"/><stop offset="1" stop-color="#06b6d4"/></linearGradient></defs>
                </svg>
              </div>
              <span class="msg-sender">AI</span>
            </div>
            <div class="msg-bubble assistant">
              <div class="typing"><span></span><span></span><span></span></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Quota bar for students -->
      <div v-if="ai.aiLimitReached.value" class="quota-bar exhausted">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
        Лимит запросов исчерпан ({{ ai.AI_LIMIT }}/{{ ai.AI_LIMIT }}). Обратитесь к администратору.
      </div>
      <div v-else-if="!ai.aiUnlimited.value && auth.user?.role === 'student'" class="quota-bar" :class="{ warn: ai.aiRemaining.value <= 2 }">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
        Осталось запросов к ИИ: <strong>{{ ai.aiRemaining.value }} / {{ ai.AI_LIMIT }}</strong>
      </div>

      <!-- File preview bar -->
      <div v-if="pendingFile" class="file-prev">
        <div class="fp-info">
          <img v-if="pendingPreview" :src="pendingPreview" class="fp-thumb" alt="preview"/>
          <span>{{pendingFile.type.startsWith('image/') ? '🖼️' : '📎'}} {{pendingFile.name}}</span>
          <span v-if="pendingFile.type.startsWith('image/')" class="fp-badge">ИИ прочитает текст</span>
        </div>
        <button class="fp-rm" @click="clearFile">×</button>
      </div>

      <!-- Input -->
      <div class="chat-inp">
        <label class="attach-lbl" title="Прикрепить изображение / файл">
          <input ref="fileInput" type="file" accept="image/*,.pdf,.doc,.docx,.txt" style="display:none" @change="onFilePick"/>
          <div class="attach-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21.44 11.05l-9.19 9.19a6 6 0 01-8.49-8.49l9.19-9.19a4 4 0 015.66 5.66l-9.2 9.19a2 2 0 01-2.83-2.83l8.49-8.48"/></svg>
          </div>
        </label>
        <input
          ref="inp"
          v-model="txt"
          class="chat-field"
          :placeholder="ai.aiLimitReached.value ? 'Лимит запросов исчерпан...' : 'Написать сообщение или спросить кое что...'"
          :disabled="loading || ai.aiLimitReached.value"
          @keydown.enter="send"
        />
        <button
          :class="['send-btn', {active: (txt.trim() || pendingFile) && !ai.aiLimitReached.value}]"
          :disabled="(!txt.trim() && !pendingFile) || loading || ai.aiLimitReached.value"
          @click="send"
        >
          <div v-if="loading" class="spinner" style="width:14px;height:14px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
          <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="currentColor"><path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/></svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick, onMounted, onUnmounted } from 'vue'
import { useAi } from '~/composables/useAi/useAi'
import { useI18n } from '~/composables/usel18n/useI18n'
import { useAuthStore } from '~/stores/auth.store'

definePageMeta({ layout: 'default' })

const ai = useAi()
const auth = useAuthStore()
const { t, lang } = useI18n()
const msgs = computed(() => ai.msgs.value)
const loading = computed(() => ai.loading.value)

const area = ref<HTMLElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const inp = ref<HTMLInputElement | null>(null)
const fileInput = ref<HTMLInputElement | null>(null)
const txt = ref('')
const pendingFile = ref<File | null>(null)
const pendingPreview = ref<string | null>(null)

const tips = [
  'Давай пообщаемся',
  'Объясни тему кратко',
  'Помоги с кодом',
  'Составь план урока',
  'Проверь мой текст',
]

const fmt = (t: string) => t
  .replace(/```(\w+)?\n?([^`]*?)```/gs, '<pre class="code-bl"><code>$2</code></pre>')
  .replace(/`([^`]+)`/g, '<code class="ic">$1</code>')
  .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
  .replace(/\n/g, '<br>')

const scroll = () => nextTick(() => {
  if (area.value) area.value.scrollTop = area.value.scrollHeight
})

const onFilePick = (e: Event) => {
  const f = (e.target as HTMLInputElement).files?.[0]
  if (!f) return
  pendingFile.value = f
  if (f.type.startsWith('image/')) {
    const reader = new FileReader()
    reader.onload = () => { pendingPreview.value = reader.result as string }
    reader.readAsDataURL(f)
  } else {
    pendingPreview.value = null
  }
  // Reset input so same file can be picked again
  if (fileInput.value) fileInput.value.value = ''
}

const clearFile = () => {
  pendingFile.value = null
  pendingPreview.value = null
}

const send = async () => {
  if ((!txt.value.trim() && !pendingFile.value) || loading.value) return
  const message = txt.value
  const file = pendingFile.value
  txt.value = ''
  pendingFile.value = null
  pendingPreview.value = null
  await ai.send(message, file)
  scroll()
}

const quick = async (t: string) => {
  // Strip emoji prefix for chip tips
  txt.value = t.replace(/^[^\s]+\s/, '')
  await send()
}

watch(() => ai.msgs.value.length, () => scroll())

// ── Neural Network Animation ──────────────────────────────────────────────────
let animId: number

interface Node {
  x: number; y: number; vx: number; vy: number
  r: number; pulse: number; phase: number
}

onMounted(() => {
  const c = canvas.value
  if (!c) return
  const ctx = c.getContext('2d')
  if (!ctx) return

  const resize = () => {
    c.width = c.offsetWidth
    c.height = c.offsetHeight
  }
  resize()
  window.addEventListener('resize', resize)

  // Create nodes — represents neurons / knowledge nodes
  const NODE_COUNT = 38
  const nodes: Node[] = Array.from({ length: NODE_COUNT }, () => ({
    x: Math.random() * c.width,
    y: Math.random() * c.height,
    vx: (Math.random() - 0.5) * 0.35,
    vy: (Math.random() - 0.5) * 0.35,
    r: 1.5 + Math.random() * 3,
    pulse: Math.random() * Math.PI * 2,
    phase: Math.random() * Math.PI * 2,
  }))

  const CONNECTION_DIST = 130
  let t = 0

  const draw = () => {
    c.width = c.offsetWidth
    c.height = c.offsetHeight
    ctx.clearRect(0, 0, c.width, c.height)

    // Subtle radial glow
    const glow = ctx.createRadialGradient(
      c.width * 0.5, c.height * 0.45, 0,
      c.width * 0.5, c.height * 0.45, c.width * 0.55
    )
    glow.addColorStop(0, 'rgba(0,177,201,0.05)')
    glow.addColorStop(0.6, 'rgba(6,182,212,0.025)')
    glow.addColorStop(1, 'rgba(0,0,0,0)')
    ctx.fillStyle = glow
    ctx.fillRect(0, 0, c.width, c.height)

    // Move nodes
    for (const n of nodes) {
      n.x += n.vx
      n.y += n.vy
      n.pulse += 0.025
      if (n.x < 0 || n.x > c.width) n.vx *= -1
      if (n.y < 0 || n.y > c.height) n.vy *= -1
    }

    // Draw connections
    for (let i = 0; i < nodes.length; i++) {
      for (let j = i + 1; j < nodes.length; j++) {
        const a = nodes[i], b = nodes[j]
        const dx = a.x - b.x, dy = a.y - b.y
        const dist = Math.sqrt(dx * dx + dy * dy)
        if (dist > CONNECTION_DIST) continue

        const alpha = (1 - dist / CONNECTION_DIST) * 0.35
        // Pulse signal along connection
        const signalPos = (Math.sin(t * 0.018 + a.phase) * 0.5 + 0.5)
        const sx = a.x + (b.x - a.x) * signalPos
        const sy = a.y + (b.y - a.y) * signalPos

        // Connection line
        const grad = ctx.createLinearGradient(a.x, a.y, b.x, b.y)
        grad.addColorStop(0, `rgba(0,177,201,${alpha})`)
        grad.addColorStop(0.5, `rgba(6,182,212,${alpha * 1.3})`)
        grad.addColorStop(1, `rgba(0,177,201,${alpha})`)
        ctx.beginPath()
        ctx.moveTo(a.x, a.y)
        ctx.lineTo(b.x, b.y)
        ctx.strokeStyle = grad
        ctx.lineWidth = 0.8
        ctx.stroke()

        // Travelling signal dot
        const sa = alpha * 2.5
        ctx.beginPath()
        ctx.arc(sx, sy, 1.8, 0, Math.PI * 2)
        ctx.fillStyle = `rgba(196,181,253,${sa})`
        ctx.shadowBlur = 8
        ctx.shadowColor = 'rgba(0,177,201,0.8)'
        ctx.fill()
        ctx.shadowBlur = 0
      }
    }

    // Draw nodes (neurons)
    for (const n of nodes) {
      const pulse = Math.sin(n.pulse) * 0.5 + 0.5
      const r = n.r * (0.85 + pulse * 0.3)
      const a = 0.4 + pulse * 0.5

      // Outer glow
      const nodeGrad = ctx.createRadialGradient(n.x, n.y, 0, n.x, n.y, r * 3.5)
      nodeGrad.addColorStop(0, `rgba(167,139,250,${a * 0.5})`)
      nodeGrad.addColorStop(1, 'rgba(167,139,250,0)')
      ctx.beginPath()
      ctx.arc(n.x, n.y, r * 3.5, 0, Math.PI * 2)
      ctx.fillStyle = nodeGrad
      ctx.fill()

      // Core dot
      ctx.beginPath()
      ctx.arc(n.x, n.y, r, 0, Math.PI * 2)
      ctx.fillStyle = `rgba(196,181,253,${a})`
      ctx.shadowBlur = 10 + pulse * 8
      ctx.shadowColor = `rgba(0,177,201,${a})`
      ctx.fill()
      ctx.shadowBlur = 0
    }

    // Floating knowledge glyphs (ΣΔΩ ∫ ∞ π)
    const glyphs = ['Σ', 'Δ', '∫', '∞', 'π', 'Ω', '√', '∂']
    for (let i = 0; i < glyphs.length; i++) {
      const x = c.width * (0.08 + i * 0.12 + Math.sin(t * 0.005 + i * 0.9) * 0.04)
      const y = c.height * (0.15 + Math.sin(t * 0.007 + i * 1.3) * 0.1 + (i % 2) * 0.55)
      const a = 0.04 + Math.sin(t * 0.008 + i) * 0.025
      ctx.font = `${13 + Math.sin(t * 0.01 + i) * 2}px serif`
      ctx.fillStyle = `rgba(167,139,250,${a})`
      ctx.fillText(glyphs[i], x, y)
    }

    t++
    animId = requestAnimationFrame(draw)
  }

  draw()

  onUnmounted(() => {
    cancelAnimationFrame(animId)
    window.removeEventListener('resize', resize)
  })
})
</script>

<style scoped src="~/assets/pages/ai.css"></style>
