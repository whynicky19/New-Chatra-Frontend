<template>
  <div class="cc" @click="$emit('open')">
    <div class="cc-cover" :style="coverBg">
      <img v-if="coverImage" :src="coverImage" class="cc-cover-img"/>
      <div class="cc-cover-overlay"></div>
      <div class="cc-title-over">{{cls.title}}</div>
      <div v-if="hwCount>0" class="cc-due-badge">{{hwCount}} due</div>
    </div>
    <div class="cc-body">
      <div class="cc-desc">{{classDesc || 'Нажмите для просмотра материалов'}}</div>
      <div class="cc-footer">
        <div class="cc-teacher">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
          {{teacherName}}
        </div>
        <button class="cc-view-btn">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 19.5A2.5 2.5 0 016.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 014 19.5v-15A2.5 2.5 0 016.5 2z"/></svg>
          View
        </button>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { computed } from 'vue'
const props = defineProps<{cls:{id:number;title:string;body:string;user_id:number};hwCount?:number}>()
defineEmits<{open:[]}>()

const gradients = [
  'linear-gradient(135deg,#1a0533,#3d0f75)',
  'linear-gradient(135deg,#0a1628,#1a3a5c)',
  'linear-gradient(135deg,#1a0a2e,#4a1060)',
  'linear-gradient(135deg,#0d1f0d,#1a4a1a)',
  'linear-gradient(135deg,#1a0a0a,#4a1a1a)',
]

const parsed = computed(() => { try { return JSON.parse(props.cls.body) } catch { return {} } })
const coverImage = computed(() => parsed.value.cover_image || null)
const classDesc = computed(() => parsed.value.description || '')
const coverBg = computed(() => coverImage.value ? {} : { background: gradients[props.cls.id % gradients.length] })
const teacherName = computed(() => {
  if (parsed.value.teacher) return parsed.value.teacher
  try { const reg = JSON.parse(localStorage.getItem('_nick_registry')||'{}'); return reg[props.cls.user_id] || 'Преподаватель' } catch { return 'Преподаватель' }
})
const hwCount = computed(() => props.hwCount || 0)
</script>

<style scoped src="~/assets/components/classCard.css"></style>
