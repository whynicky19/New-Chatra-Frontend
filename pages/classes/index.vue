<template>
  <div class="pg anim-in">
    <div class="pg-body">
      <div class="content-area">
        <!-- Header -->
        <div class="pg-head">
          <div class="pg-head-left">
            <h1 class="pg-title">{{ t('classes.catalog') }}</h1>
            <p class="pg-sub">{{ lang === 'ru' ? 'Исследуйте новые горизонты знаний с нашими интерактивными модулями.' : 'Explore new horizons of knowledge with our interactive modules.' }}</p>
          </div>
          <div class="pg-head-r">
            <button v-if="auth.isTeacher" class="btn btn-outline-teal" @click="showCreate=true">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
              {{ t('classes.create') }}
            </button>
            <button class="btn btn-teal" @click="showJoin=true">
              {{ t('classes.join_code') }}
            </button>
          </div>
        </div>

        <!-- Grid -->
        <div class="classes-grid">
          <template v-if="loading">
            <div v-for="n in 6" :key="n" class="class-card skeleton-card">
              <div class="skel-cover"></div>
              <div class="skel-body">
                <div class="skel-line skel-title"></div>
                <div class="skel-line skel-desc"></div>
                <div class="skel-line skel-desc short"></div>
                <div class="skel-line skel-meta"></div>
              </div>
            </div>
          </template>

          <div v-if="!loading && !visibleClasses.length" class="empty-state" style="grid-column:1/-1">
            <div class="es-icon-wrap">
              <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.3"><path d="M2 3h6a4 4 0 014 4v14a3 3 0 00-3-3H2z"/><path d="M22 3h-6a4 4 0 00-4 4v14a3 3 0 013-3h7z"/></svg>
            </div>
            <div class="es-title">{{ auth.isTeacher ? t('classes.no_classes_teacher') : t('classes.no_classes') }}</div>
            <div class="es-sub">{{ auth.isTeacher ? t('classes.no_classes_sub_teacher') : t('classes.no_classes_sub_student') }}</div>
            <button class="btn btn-teal es-btn" @click="showJoin=true">{{ t('classes.join_code') }}</button>
          </div>

          <template v-if="!loading && visibleClasses.length">
            <!-- Class cards -->
            <div v-for="cls in visibleClasses" :key="cls.id" class="class-card" @click="goClass(cls.id)">
              <!-- Cover image -->
              <div class="card-cover" :style="cls.cover_image ? {} : {background: coverGrad(cls.id)}">
                <img v-if="cls.cover_image" :src="cls.cover_image" class="card-cover-img" loading="lazy" decoding="async"/>
                <!-- Code chip for teachers -->
                <div v-if="auth.isTeacher || auth.isAdmin" class="card-code-chip" @click.stop="copyClassCode(cls.id)">
                  <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1"/></svg>
                  {{ codeFor(cls.id) }}
                </div>
              </div>

              <!-- Card body -->
              <div class="card-body">
                <div class="card-title-row">
                  <h3 class="card-name">{{ cls.title }}</h3>
                  <div class="card-subject-icon">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M2 3h6a4 4 0 014 4v14a3 3 0 00-3-3H2z"/><path d="M22 3h-6a4 4 0 00-4 4v14a3 3 0 013-3h7z"/></svg>
                  </div>
                </div>
                <p class="card-desc">{{ cls.description || (lang==='ru' ? 'Нажмите для просмотра' : 'Click to view') }}</p>
                <div class="card-meta">
                  <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
                  {{ lectureCount(cls.id) }} {{ lang === 'ru' ? 'уроков' : 'lessons' }}
                </div>
                <div class="card-footer">
                  <div class="card-action-row">
                    <button v-if="!auth.isTeacher" class="card-action-btn" @click.stop="goClass(cls.id)">
                      {{ getActionLabel(cls) }} →
                    </button>
                    <button v-else class="card-action-btn" @click.stop="goClass(cls.id)">
                      {{ lang === 'ru' ? 'Открыть курс' : 'Open course' }} →
                    </button>
                    <div class="card-controls">
                      <button v-if="!auth.isTeacher" class="ctrl-btn" @click.stop="leaveClass(cls.id)" :title="t('classes.left')">
                        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 01-2-2V5a2 2 0 012-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
                      </button>
                      <button v-if="auth.isTeacher" class="ctrl-btn ctrl-del" @click.stop="confirmDelete(cls)">
                        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 01-2 2H8a2 2 0 01-2-2L5 6"/></svg>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Add new class card — STUDENTS ONLY -->
            <div v-if="!auth.isTeacher && !auth.isAdmin" class="class-card add-card" @click="showJoin=true">
              <div class="add-card-inner">
                <div class="add-plus">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M5 12h14"/></svg>
                </div>
                <div class="add-title">{{ lang === 'ru' ? 'Добавить новый предмет в программу' : 'Add new subject to program' }}</div>
                <div class="add-sub">{{ lang === 'ru' ? 'Персонализируйте своё обучение' : 'Personalize your learning' }}</div>
              </div>
            </div>
          </template>
        </div>

        <!-- Upcoming deadlines -->
        <div v-if="upcomingAssignments.length" class="deadlines-section">
          <div class="deadlines-label">{{ t('classes.upcoming') }}</div>
          <div class="deadlines-list">
            <div v-for="a in upcomingAssignments" :key="a.id" class="deadline-item">
              <div class="deadline-icon">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/></svg>
              </div>
              <div class="deadline-info">
                <div class="deadline-title">{{ a.title }}</div>
                <div class="deadline-time">{{ fmtDeadline(a.deadline) }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Create modal -->
    <CreateClassModal v-if="showCreate" @close="showCreate=false" @created="onCreated"/>

    <!-- Join by code modal -->
    <div v-if="showJoin" class="overlay" @click.self="showJoin=false">
      <div class="modal anim-scale join-modal">
        <div class="modal-head">
          <span class="modal-title">{{ t('classes.join_title') }}</span>
          <button class="btn btn-icon btn-ghost" @click="showJoin=false">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
          </button>
        </div>
        <div class="join-icon-wrap">
          <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="11" width="18" height="11" rx="2"/><path d="M7 11V7a5 5 0 0110 0v4"/></svg>
        </div>
        <p class="join-hint">{{ t('classes.join_hint') }}</p>
        <div class="frow">
          <div class="code-boxes">
            <input v-for="(_, i) in 6" :key="i" :ref="el => { if(el) codeRefs[i] = el as HTMLInputElement }" class="code-box" maxlength="1" :value="codeChars[i]" @input="onCodeInput($event, i)" @keydown="onCodeKey($event, i)" @paste="onCodePaste" inputmode="text" autocomplete="off" style="text-transform:uppercase"/>
          </div>
          <div v-if="joinError" class="join-err">{{joinError}}</div>
          <div v-if="foundClass" class="join-found">
            <div class="found-cover" :style="foundClass.cover_image ? `background-image:url(${foundClass.cover_image})` : `background:${coverGrad(foundClass.id)}`">
              <div class="found-overlay"></div>
              <span class="found-name">{{foundClass.title}}</span>
            </div>
            <div class="found-meta">{{foundClass.teacher || (lang==='ru'?'Преподаватель':'Teacher')}}</div>
          </div>
        </div>
        <div class="modal-foot">
          <button class="btn btn-white" @click="showJoin=false">{{ t('general.cancel') }}</button>
          <button class="btn btn-teal" :disabled="joinCode.length<6||joining" @click="joinClass">
            <div v-if="joining" class="spinner" style="width:13px;height:13px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
            <span v-else>{{ t('classes.join') }}</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Delete confirm modal -->
    <div v-if="deletingClass" class="overlay" @click.self="deletingClass=null">
      <div class="modal anim-scale" style="max-width:400px">
        <div class="modal-head">
          <span class="modal-title">{{ t('classes.delete_title') }}</span>
          <button class="btn btn-icon btn-ghost" @click="deletingClass=null">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
          </button>
        </div>
        <div class="del-body">
          <div class="del-icon"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 01-2 2H8a2 2 0 01-2-2L5 6"/></svg></div>
          <p class="del-text">{{ t('classes.delete_confirm') }} <strong>«{{deletingClass.title}}»</strong>? {{ t('classes.delete_warn') }}</p>
        </div>
        <div class="modal-foot">
          <button class="btn btn-white" @click="deletingClass=null">{{ t('general.cancel') }}</button>
          <button class="btn btn-danger" :disabled="deleting" @click="doDelete">
            <div v-if="deleting" class="spinner" style="width:13px;height:13px;border-width:2px;border-color:rgba(220,38,38,.3);border-top-color:#dc2626"></div>
            <span v-else>{{ t('classes.delete') }}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from '#app'
import { useAuthStore } from '~/stores/auth.store'
import { usePostsSvc } from '~/services/posts'
import { useToast } from '~/composables/useToast/useToast'
import { useI18n } from '~/composables/usel18n/useI18n'
definePageMeta({ layout: 'default' })
const auth = useAuthStore(); const postsSvc = usePostsSvc(); const toast = useToast(); const router = useRouter()
const { t, lang } = useI18n()
const allPosts = ref<any[]>([]); const loading = ref(true); const showCreate = ref(false)
const showJoin = ref(false); const joining = ref(false); const joinError = ref('')
const deletingClass = ref<any>(null); const deleting = ref(false)

const codeChars = ref<string[]>(['','','','','',''])
const codeRefs = ref<HTMLInputElement[]>([])
const joinCode = computed(() => codeChars.value.join(''))

const onCodeInput = (e: Event, i: number) => {
  const val = (e.target as HTMLInputElement).value.toUpperCase().replace(/[^A-Z0-9]/g,'')
  codeChars.value[i] = val.slice(-1)
  if (val && i < 5) codeRefs.value[i+1]?.focus()
  joinError.value = ''
}
const onCodeKey = (e: KeyboardEvent, i: number) => {
  if (e.key === 'Backspace' && !codeChars.value[i] && i > 0) { codeChars.value[i-1] = ''; codeRefs.value[i-1]?.focus() }
  if (e.key === 'Enter' && joinCode.value.length === 6) joinClass()
}
const onCodePaste = (e: ClipboardEvent) => {
  e.preventDefault()
  const text = e.clipboardData?.getData('text').toUpperCase().replace(/[^A-Z0-9]/g,'').slice(0,6) || ''
  text.split('').forEach((c, i) => { if (i < 6) codeChars.value[i] = c })
  codeRefs.value[Math.min(text.length, 5)]?.focus()
}

const joinedKey = computed(() => `_joined_${auth.user?.id||0}`)
const joinedIds = ref<number[]>([])
const loadJoined = () => { try { joinedIds.value = JSON.parse(localStorage.getItem(joinedKey.value)||'[]') } catch { joinedIds.value=[] } }
const saveJoined = () => localStorage.setItem(joinedKey.value, JSON.stringify(joinedIds.value))

const covers = [
  'linear-gradient(135deg,#006475,#009aaf)',
  'linear-gradient(135deg,#0c4a6e,#0369a1)',
  'linear-gradient(135deg,#134e4a,#0d9488)',
  'linear-gradient(135deg,#312e81,#4338ca)',
  'linear-gradient(135deg,#1e3a5f,#2563eb)'
]
const coverGrad = (id: number) => covers[id % covers.length]

const allClasses = computed(() =>
  allPosts.value
    .filter(p => { try { const b=JSON.parse(p.body); return b.type==='class' } catch { return false } })
    .map(p => { try { const b=JSON.parse(p.body); return {...p,...b,title:p.title,cover_image:b.cover_image||null,description:b.description||''} } catch { return p } })
)
const visibleClasses = computed(() => auth.isAdmin ? allClasses.value : allClasses.value.filter(c => joinedIds.value.includes(c.id)))
const lectureCount = (classId: number) => allPosts.value.filter(p => p.title?.startsWith(`[LECTURE][${classId}]`)).length

const getActionLabel = (cls: any) => {
  const count = lectureCount(cls.id)
  if (count === 0) return lang.value === 'ru' ? 'Начать обучение' : 'Start learning'
  return lang.value === 'ru' ? 'Продолжить обучение' : 'Continue learning'
}

const codeFor = (id: number) => {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'; let code = ''; let n = id*1337+42
  for(let i=0;i<6;i++){code+=chars[n%chars.length];n=Math.floor(n/chars.length)+id*7}
  return code.slice(0,6)
}
const foundClass = computed(() => {
  if (joinCode.value.length < 6) return null
  return allClasses.value.find(c => codeFor(c.id) === joinCode.value.toUpperCase()) || null
})

const upcomingAssignments = computed(() => {
  const now = new Date()
  return allPosts.value
    .filter(p => { try { const b=JSON.parse(p.body); return b.type==='assignment' && b.deadline && new Date(b.deadline) > now } catch { return false } })
    .map(p => { try { const b=JSON.parse(p.body); return {...b, id:p.id, title:p.title} } catch { return null } })
    .filter(Boolean).slice(0, 3)
})
const fmtDeadline = (d: string) => {
  if (!d) return ''
  try {
    const diff = new Date(d).getTime() - Date.now()
    const hrs = Math.round(diff / 3600000)
    if (hrs < 24) {
      const key = hrs === 1 ? 'deadline.in_hours_1' : 'deadline.in_hours'
      return t(key).replace('{n}', String(hrs))
    }
    return `${t('deadline.tomorrow')}, ${new Date(d).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`
  } catch { return d }
}

const joinClass = () => {
  joining.value = true; joinError.value = ''
  const found = foundClass.value
  setTimeout(() => {
    if (!found) { joinError.value = t('classes.not_found'); joining.value=false; return }
    if (!joinedIds.value.includes(found.id)) { joinedIds.value.push(found.id); saveJoined() }
    joining.value=false; showJoin.value=false; codeChars.value=['','','','','','']
    toast.ok(`${t('classes.joined')} ${found.title}`)
  }, 400)
}
const leaveClass = (id: number) => { joinedIds.value = joinedIds.value.filter(i => i !== id); saveJoined(); toast.ok(t('classes.left_ok')) }
const confirmDelete = (cls: any) => { deletingClass.value = cls }
const doDelete = async () => {
  if (!deletingClass.value) return
  deleting.value = true
  try { await postsSvc.remove(deletingClass.value.id); allPosts.value = allPosts.value.filter(p => p.id !== deletingClass.value.id); toast.ok(t('general.delete')); deletingClass.value = null }
  catch(e: any) { toast.err(e?.response?.data?.detail || t('general.error')) }
  finally { deleting.value = false }
}
const goClass = (id: number) => router.push(`/classes/${id}`)
const copyClassCode = (id: number) => {
  const code = codeFor(id)
  navigator.clipboard?.writeText(code).then(() => toast.ok(`Код скопирован: ${code}`)).catch(() => toast.ok(`Код: ${code}`))
}
const onCreated = async (cls: any) => { showCreate.value=false; await load(); if (cls?.id && !joinedIds.value.includes(cls.id)) { joinedIds.value.push(cls.id); saveJoined() } }
const load = async () => { loading.value=true; try { allPosts.value=await postsSvc.list() } catch { toast.err(t('general.error')) } finally { loading.value=false } }
watch(() => auth.user?.id, (id) => { if (id) loadJoined() }, { immediate: true })
onMounted(()=>{ load() })
</script>

<style scoped src="~/assets/pages/classes.css"></style>
