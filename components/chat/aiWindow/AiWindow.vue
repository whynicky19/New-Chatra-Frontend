<template>
  <div class="ai">
    <div class="ai-head">
      <div class="ai-head-l">
        <div class="ai-av">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
        </div>
        <div>
          <div class="ai-head-title">ИИ Ассистент</div>
          <div class="ai-head-sub">
            <span class="ai-dot"></span>
            {{ loading ? 'Думает...' : 'Онлайн · читает рукописи' }}
          </div>
        </div>
      </div>
      <button class="btn btn-ghost btn-sm" @click="ai.clear()">Очистить</button>
    </div>

    <!-- Quota bar for students -->
    <div v-if="ai.aiLimitReached.value" class="ai-quota-block exhausted">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
      Лимит запросов исчерпан ({{ ai.AI_LIMIT }}/{{ ai.AI_LIMIT }}). Обратитесь к администратору.
    </div>
    <div v-else-if="!ai.aiUnlimited.value && auth.user?.role === 'student'" class="ai-quota-block" :class="{ warn: ai.aiRemaining.value <= 2 }">
      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
      Осталось запросов к ИИ: <strong>{{ ai.aiRemaining.value }} / {{ ai.AI_LIMIT }}</strong>
    </div>

    <div ref="area" class="ai-area">
      <div v-if="!msgs.length" class="ai-welcome anim-in">
        <div class="ai-welcome-icon">
          <svg width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
        </div>
        <h3 class="ai-welcome-title">AI Ассистент</h3>
        <p class="ai-welcome-desc">Задайте вопрос или прикрепите фото с рукописью — прочитаю и отвечу</p>
        <div class="ai-chips stagger">
          <button v-for="t in tips" :key="t" class="ai-chip" @click="quick(t)">{{ t }}</button>
        </div>
      </div>

      <div v-else class="ai-msgs">
        <div v-for="m in msgs" :key="m.id" :class="['ai-msg', m.role]">
          <div :class="['ai-av-m', m.role === 'user' ? 'u' : 'b']">
            <svg v-if="m.role === 'assistant'" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
            <span v-else>{{ uInit }}</span>
          </div>
          <div class="ai-mc">
            <img v-if="m.imagePreview" :src="m.imagePreview" class="ai-img-prev" alt="uploaded"/>
            <div class="ai-bbl" v-html="fmt(m.text)"></div>
            <div class="ai-ts">{{ ts(m.ts) }}</div>
          </div>
        </div>
        <div v-if="loading" class="ai-msg assistant">
          <div class="ai-av-m b">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
          </div>
          <div class="ai-mc"><div class="typing"><span></span><span></span><span></span></div></div>
        </div>
      </div>
    </div>

    <!-- File preview -->
    <div v-if="pendingFile" class="ai-file-prev">
      <img v-if="pendingPreview" :src="pendingPreview" class="ai-fp-thumb" alt="preview"/>
      <span>{{ pendingFile.type.startsWith('image/') ? '🖼️' : '📎' }} {{ pendingFile.name }}</span>
      <span v-if="pendingFile.type.startsWith('image/')" class="ai-fp-badge">распознаю текст</span>
      <button class="ai-fp-rm" @click="clearFile">×</button>
    </div>

    <div class="ai-inp">
      <label class="ai-attach" title="Прикрепить изображение">
        <input ref="fileInp" type="file" accept="image/*,.pdf,.txt" style="display:none" @change="onFilePick"/>
        <div class="ai-attach-icon">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21.44 11.05l-9.19 9.19a6 6 0 01-8.49-8.49l9.19-9.19a4 4 0 015.66 5.66l-9.2 9.19a2 2 0 01-2.83-2.83l8.49-8.48"/></svg>
        </div>
      </label>
      <textarea
        ref="ta"
        v-model="txt"
        class="ai-field"
        :placeholder="ai.aiLimitReached.value ? 'Лимит запросов исчерпан...' : 'Написать сообщение...'"
        rows="1"
        :disabled="loading || ai.aiLimitReached.value"
        @keydown.enter.exact.prevent="send"
        @input="resize"
      ></textarea>
      <button :class="['send-btn', { active: (txt.trim() || pendingFile) && !ai.aiLimitReached.value }]" :disabled="(!txt.trim() && !pendingFile) || loading || ai.aiLimitReached.value" @click="send">
        <div v-if="loading" class="spinner" style="width:14px;height:14px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
        <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/></svg>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue'
import { useAuthStore } from '~/stores/auth.store'
import { useAi } from '~/composables/useAi/useAi'

const auth = useAuthStore()
const ai = useAi()

const msgs = computed(() => ai.msgs.value)
const loading = computed(() => ai.loading.value)

const area = ref<HTMLElement | null>(null)
const ta = ref<HTMLTextAreaElement | null>(null)
const fileInp = ref<HTMLInputElement | null>(null)
const txt = ref('')
const pendingFile = ref<File | null>(null)
const pendingPreview = ref<string | null>(null)

const uInit = computed(() => (auth.nickname || auth.user?.email || '?')[0]?.toUpperCase())
const tips = ['📸 Фото задания — прочитаю', 'Объясни тему кратко', 'Помоги с домашним заданием', 'Составь план урока', 'Проверь мой текст']
const ts = (d: Date) => d.toLocaleTimeString('ru-RU', { hour: '2-digit', minute: '2-digit' })

const fmt = (t: string) => t
  .replace(/```(\w+)?\n?([^`]*?)```/g, '<pre class="code-bl"><code>$2</code></pre>')
  .replace(/`([^`]+)`/g, '<code class="ic">$1</code>')
  .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
  .replace(/\n/g, '<br>')

const scroll = () => nextTick(() => { if (area.value) area.value.scrollTop = area.value.scrollHeight })
const resize = () => { if (!ta.value) return; ta.value.style.height = 'auto'; ta.value.style.height = Math.min(ta.value.scrollHeight, 120) + 'px' }

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
  if (fileInp.value) fileInp.value.value = ''
}

const clearFile = () => { pendingFile.value = null; pendingPreview.value = null }

const send = async () => {
  if ((!txt.value.trim() && !pendingFile.value) || loading.value) return
  const message = txt.value
  const file = pendingFile.value
  txt.value = ''
  pendingFile.value = null
  pendingPreview.value = null
  if (ta.value) ta.value.style.height = 'auto'
  await ai.send(message, file)
  scroll()
}

const quick = async (t: string) => {
  txt.value = t.replace(/^[^\s]+\s/, '')
  await send()
}

watch(() => ai.msgs.value.length, () => scroll())
</script>

<style scoped src="~/assets/components/ai-window.css"></style>
