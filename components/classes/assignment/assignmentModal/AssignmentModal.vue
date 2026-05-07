<template>
  <div class="overlay" @click.self="$emit('close')">
    <div class="am-modal anim-scale">

      <!-- Header -->
      <div class="am-head">
        <div class="am-head-l">
          <div class="am-ico"><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg></div>
          <div>
            <div class="am-title">{{ assignment.title }}</div>
            <div class="am-badges">
              <span class="badge-score">{{ assignment.max_score }} баллов</span>
              <span v-if="deadlineStr" :class="['badge-due', isOverdue ? 'overdue' : '']">
                <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
                {{ deadlineStr }}
              </span>
            </div>
          </div>
        </div>
        <button class="btn btn-icon btn-ghost" @click="$emit('close')"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg></button>
      </div>

      <!-- Tabs -->
      <div class="am-tabs">
        <button :class="['am-tab', { active: tab === 'info' }]" @click="tab = 'info'">Задание</button>
        <button v-if="canSeeSubmissions" :class="['am-tab', { active: tab === 'submissions' }]" @click="tab = 'submissions'; loadSubs()">
          Работы <span v-if="submissions.length" class="tab-count">{{ submissions.length }}</span>
        </button>
        <button v-if="!canSeeSubmissions" :class="['am-tab', { active: tab === 'submit' }]" @click="tab = 'submit'">
          {{ mySubmission ? 'Моя работа' : 'Сдать' }}
        </button>
      </div>

      <!-- ═══ INFO TAB ═══ -->
      <div v-if="tab === 'info'" class="am-body">
        <div v-if="assignment.description" class="desc-block">{{ assignment.description }}</div>

        <!-- Assignment files (if teacher attached) -->
        <div v-if="assignmentFiles.length" class="section">
          <div class="section-label">Файлы задания</div>
          <div class="files-row">
            <a v-for="f in assignmentFiles" :key="f.url" :href="f.url" target="_blank" class="file-chip">
              <span>{{ getEmoji(f.url) }}</span> {{ f.name }}
            </a>
          </div>
        </div>

        <div v-if="canSeeSubmissions" class="section">
          <div class="section-label">Критерии оценивания</div>
          <div class="criteria-list">
            <div v-for="c in parsedCriteria" :key="c.name" class="criterion">
              <div class="criterion-top">
                <span class="criterion-name">{{ c.name }}</span>
                <span class="criterion-pts">{{ c.weight }} / {{ assignment.max_score }}</span>
              </div>
              <div v-if="c.description" class="criterion-desc">{{ c.description }}</div>
              <div class="criterion-bar"><div class="criterion-bar-fill" :style="{ width: (c.weight / assignment.max_score * 100) + '%' }"></div></div>
            </div>
          </div>
        </div>
      </div>

      <!-- ═══ SUBMIT TAB (student) ═══ -->
      <div v-if="tab === 'submit' && !canSeeSubmissions" class="am-body">

        <!-- Already submitted -->
        <div v-if="mySubmission" class="submitted-panel">
          <div class="sub-status-bar">
            <div :class="['sub-status-chip', mySubmission.status]">
              <svg v-if="['submitted','graded'].includes(mySubmission.status)" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="20 6 9 17 4 12"/></svg>
              <svg v-else-if="mySubmission.status === 'grading'" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
              <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
              {{ statusLabel(mySubmission.status) }}
            </div>
            <span v-if="mySubmission.variant_number" class="variant-badge">Вариант {{ mySubmission.variant_number }}</span>
            <span class="sub-date">{{ fmtDate(mySubmission.submitted_at) }}</span>
          </div>

          <div v-if="mySubmission.text_content" class="preview-block">
            <div class="preview-label">Ваш ответ</div>
            <div class="preview-text">{{ mySubmission.text_content }}</div>
          </div>

          <div v-if="mySubmission.file_url || parsedSubmittedUrls.length" class="sub-file">
            <a v-for="url in (parsedSubmittedUrls.length ? parsedSubmittedUrls : [mySubmission.file_url])" :key="url" :href="url" target="_blank" class="file-chip">
              <span>{{ getEmoji(url) }}</span> {{ getFileName(url) }}
            </a>
          </div>

          <!-- Grade card -->
          <div v-if="mySubmission.grade" class="grade-card">
            <div class="grade-card-top">
              <div class="grade-score"><span class="grade-num">{{ mySubmission.grade.score }}</span><span class="grade-denom">/ {{ assignment.max_score }}</span></div>
              <div class="grade-meta">
                <div class="grade-pct">{{ Math.round(mySubmission.grade.score / assignment.max_score * 100) }}%</div>
                <div class="grade-by-tag">
                  <svg v-if="mySubmission.grade.graded_by === 'ai'" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
                  <svg v-else width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                  {{ mySubmission.grade.graded_by === 'ai' ? 'ИИ' : 'Учитель' }}
                </div>
              </div>
            </div>
            <div v-if="mySubmission.grade.feedback" class="grade-feedback">
              <div class="section-label">Фидбек</div>
              <div class="feedback-text">{{ mySubmission.grade.feedback }}</div>
            </div>
            <div v-if="parsedCriteriaScores" class="grade-criteria">
              <div class="section-label">По критериям</div>
              <div v-for="cs in parsedCriteriaScores" :key="cs.name" class="cs-item">
                <div class="cs-top"><span class="cs-name">{{ cs.name }}</span><span class="cs-pts">{{ cs.score }} / {{ cs.max }}</span></div>
                <div v-if="cs.comment" class="cs-comment">{{ cs.comment }}</div>
                <div class="cs-bar"><div class="cs-bar-fill" :style="{ width: (cs.score / cs.max * 100) + '%' }"></div></div>
              </div>
            </div>
          </div>

          <div v-else-if="mySubmission.status === 'grading'" class="grading-pending">
            <div class="grading-dots"><span></span><span></span><span></span></div>
            ИИ проверяет вашу работу...
          </div>

          <!-- Retract button (only if not graded) -->
          <button v-if="mySubmission.status !== 'graded'" class="btn btn-ghost retract-btn" :disabled="retracting" @click="retract">
            <div v-if="retracting" class="spinner"></div>
            <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 102.13-9.36L1 10"/></svg>
            {{ retracting ? 'Отмена...' : 'Отозвать и сдать заново' }}
          </button>
        </div>

        <!-- Submit form -->
        <div v-else class="submit-form">
          <div v-if="isOverdue" class="overdue-warn">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
            Дедлайн истёк — работа будет помечена как просроченная
          </div>

          <!-- Assignment files hint -->
          <div v-if="assignmentFiles.length" class="task-files-hint">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
            Файлы задания:
            <a v-for="f in assignmentFiles" :key="f.url" :href="f.url" target="_blank" class="file-chip small">{{ getEmoji(f.url) }} {{ f.name }}</a>
          </div>

          <!-- Variant selector -->
          <div v-if="assignmentVariants.length" class="field">
            <label class="field-label">Ваш вариант *</label>
            <div class="variant-chips">
              <button v-for="v in assignmentVariants" :key="v.id"
                :class="['variant-chip', { active: form.variantNumber === v.variant_number }]"
                @click="form.variantNumber = v.variant_number">
                Вариант {{ v.variant_number }}{{ v.title ? ': ' + v.title : '' }}
              </button>
            </div>
            <div v-if="assignmentVariants.length && !form.variantNumber" class="field-hint-warn">Выберите вариант перед сдачей</div>
          </div>

          <div class="field">
            <label class="field-label">Текст ответа</label>
            <textarea v-model="form.text" class="inp inp-ta" rows="6" placeholder="Напишите ваш ответ..."></textarea>
          </div>

          <div class="field">
            <label class="field-label">Прикрепить файлы</label>
            <div class="file-drop" :class="{ 'has-file': submittedFiles.length }" @click="fileInputEl?.click()" @dragover.prevent @drop.prevent="onDrop">
              <input ref="fileInputEl" type="file" style="display:none" multiple @change="onFileSelect" />
              <template v-if="!submittedFiles.length">
                <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" style="color:var(--text4)"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
                <span class="drop-text">Перетащите или <strong>нажмите для выбора</strong></span>
                <span class="drop-hint">PDF, DOCX, Word, Excel, TXT, изображения — можно несколько</span>
              </template>
              <template v-else>
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--green)"><polyline points="20 6 9 17 4 12"/></svg>
                <span style="font-size:13px;font-weight:600;color:var(--text1)">{{ submittedFiles.length }} файл(ов) выбрано</span>
                <button class="btn btn-ghost btn-sm" @click.stop="clearFiles">Убрать все</button>
              </template>
            </div>
            <div v-if="submittedFiles.length" class="attached-files-list">
              <div v-for="(f, i) in submittedFiles" :key="i" class="attached-file-row">
                <span>{{ getEmoji(f.name) }}</span>
                <span class="af-name">{{ f.name }}</span>
                <span class="af-size">{{ fileSz(f) }}</span>
                <button class="af-rm" @click="submittedFiles.splice(i,1)">×</button>
              </div>
            </div>
            <div v-if="uploading" class="upload-prog-sm">
              <div class="upload-bar-sm" :style="{ width: uploadPctSub + '%' }"></div>
              <span>Загрузка {{ uploadIdxSub }}/{{ submittedFiles.length }}...</span>
            </div>
          </div>

          <button class="btn btn-purple btn-full" :disabled="!canSubmit || submitting" @click="doSubmit">
            <div v-if="submitting" class="spinner"></div>
            <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 2L11 13"/><path d="M22 2L15 22 11 13 2 9l20-7z"/></svg>
            {{ submitting ? (uploading ? 'Загрузка файла...' : 'Отправка...') : 'Сдать работу' }}
          </button>
        </div>
      </div>

      <!-- ═══ SUBMISSIONS TAB (teacher) ═══ -->
      <div v-if="tab === 'submissions' && canSeeSubmissions" class="am-body">
        <div v-if="loadingSubs" class="load-center"><div class="spinner" style="width:24px;height:24px;border-width:3px;border-color:var(--border2);border-top-color:var(--teal)"></div></div>

        <!-- Detail view -->
        <div v-else-if="activeSub" class="sub-detail">
          <button class="back-sub-btn" @click="activeSub = null">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M19 12H5M12 19l-7-7 7-7"/></svg>
            Назад к списку
          </button>
          <div class="sub-detail-header">
            <div class="sub-av-lg">{{ getStudentInitials(activeSub.student_id) }}</div>
            <div>
              <div class="sub-student-name">{{ getStudentName(activeSub.student_id) }}</div>
              <div class="sub-student-date" style="display:flex;align-items:center;gap:8px">
                {{ fmtDate(activeSub.submitted_at) }}
                <span v-if="activeSub.variant_number" class="variant-badge">Вариант {{ activeSub.variant_number }}</span>
              </div>
            </div>
          </div>

          <div v-if="activeSub.text_content" class="sub-text-section">
            <div class="section-label">Ответ студента</div>
            <div class="sub-text">{{ activeSub.text_content }}</div>
          </div>

          <div v-if="activeSub.file_url || parsedActiveUrls.length" class="sub-file-section">
            <div class="section-label">Прикреплённые файлы</div>
            <a v-for="url in (parsedActiveUrls.length ? parsedActiveUrls : [activeSub.file_url])" :key="url" :href="url" target="_blank" class="file-chip">
              <span>{{ getEmoji(url) }}</span> {{ getFileName(url) }}
            </a>
          </div>

          <!-- Existing grade -->
          <div v-if="activeSub.grade" class="grade-card">
            <div class="grade-card-top">
              <div class="grade-score"><span class="grade-num">{{ activeSub.grade.score }}</span><span class="grade-denom">/ {{ assignment.max_score }}</span></div>
              <div class="grade-by-tag">
                <svg v-if="activeSub.grade.graded_by === 'ai'" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
                {{ activeSub.grade.graded_by === 'ai' ? '🤖 ИИ-проверка' : '👨‍🏫 Учитель' }}
              </div>
            </div>
            <div v-if="activeSub.grade.feedback" class="grade-feedback">
              <div class="section-label">Фидбек</div>
              <div class="feedback-text">{{ activeSub.grade.feedback }}</div>
            </div>
            <div v-if="parsedActiveScores" class="grade-criteria">
              <div class="section-label">По критериям</div>
              <div v-for="cs in parsedActiveScores" :key="cs.name" class="cs-item">
                <div class="cs-top"><span class="cs-name">{{ cs.name }}</span><span class="cs-pts">{{ cs.score }} / {{ cs.max }}</span></div>
                <div v-if="cs.comment" class="cs-comment">{{ cs.comment }}</div>
                <div class="cs-bar"><div class="cs-bar-fill" :style="{ width: (cs.score / cs.max * 100) + '%' }"></div></div>
              </div>
            </div>
          </div>

          <div class="grade-actions">
            <button class="btn btn-purple" :disabled="grading" @click="runAiGrade">
              <div v-if="grading" class="spinner"></div>
              <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
              {{ grading ? 'ИИ читает файл и проверяет...' : (activeSub.grade ? 'Перепроверить ИИ' : 'Проверить ИИ') }}
            </button>
          </div>
        </div>

        <!-- List view -->
        <div v-else>
          <div v-if="!submissions.length" class="empty-block">
            <svg width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2" opacity="0.25"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/></svg>
            Работы ещё не сданы
          </div>
          <div v-else>
            <div class="subs-stats">
              <div class="stat-chip"><span class="stat-n">{{ submissions.length }}</span><span class="stat-l">Всего</span></div>
              <div class="stat-chip ok"><span class="stat-n">{{ submissions.filter(s=>s.status==='graded').length }}</span><span class="stat-l">Проверено</span></div>
              <div class="stat-chip wait"><span class="stat-n">{{ submissions.filter(s=>s.status==='submitted' || s.status==='late').length }}</span><span class="stat-l">Ожидают</span></div>
            </div>
            <button
              v-if="submissions.filter(s=>s.status==='submitted'||s.status==='late').length > 0"
              class="btn-bulk-grade"
              :disabled="bulkGrading"
              @click="runBulkAiGrade"
            >
              <svg v-if="!bulkGrading" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
              <div v-else class="spinner" style="width:12px;height:12px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
              {{ bulkGrading ? `Проверяю ${bulkDone}/${bulkTotal}...` : `⚡ Проверить все ожидающие (${submissions.filter(s=>s.status==='submitted'||s.status==='late').length})` }}
            </button>
            <div class="subs-search">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
              <input v-model="searchQuery" class="subs-search-inp" type="text" placeholder="Поиск по ФИО студента..." />
              <button v-if="searchQuery" class="subs-search-clear" @click="searchQuery = ''">×</button>
            </div>
            <div v-if="filteredSubmissions.length === 0 && searchQuery" class="empty-block" style="padding:20px">
              Студент не найден
            </div>
            <div class="subs-list">
              <div v-for="s in filteredSubmissions" :key="s.id" class="sub-row" @click="activeSub = s">
                <div class="sub-av">{{ getStudentInitials(s.student_id) }}</div>
                <div class="sub-info">
                  <div class="sub-student">{{ getStudentName(s.student_id) }}</div>
                  <div class="sub-meta">
                    <span>{{ fmtDate(s.submitted_at) }}</span>
                    <span v-if="s.variant_number" class="variant-badge">В{{ s.variant_number }}</span>
                    <span v-if="s.text_content" class="sub-tag">✍️</span>
                    <span v-if="s.file_urls || s.file_url" class="sub-tag">{{ getEmoji(s.file_url || '') }} {{ parseFileUrls(s.file_urls).length > 1 ? parseFileUrls(s.file_urls).length + ' файла' : '' }}</span>
                  </div>
                </div>
                <div class="sub-right">
                  <span v-if="s.grade" class="grade-pill">{{ s.grade.score }}/{{ assignment.max_score }}</span>
                  <span :class="['status-mini', s.status]">{{ statusLabel(s.status) }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useAssignmentsSvc } from '~/services/assignments'
import { useUploadSvc } from '~/services/uploads'
import { useUsersSvc } from '~/services/users'
import { useToast } from '~/composables/useToast/useToast'
import { useAuthStore } from '~/stores/auth.store'
import type { Assignment, Submission, Variant } from '~/services/assignments'

const props = defineProps<{ assignment: Assignment; isTeacher?: boolean }>()
const emit = defineEmits(['close', 'submitted'])

const auth = useAuthStore()
const canSeeSubmissions = computed(() => props.isTeacher || auth.isTeacher)

const svc = useAssignmentsSvc()
const uploadSvc = useUploadSvc()
const usersSvc = useUsersSvc()
const toast = useToast()

const studentMap = ref<Record<number, string>>({})

const tab = ref<'info'|'submit'|'submissions'>('info')
const searchQuery = ref('')
const mySubmission = ref<Submission|null>(null)
const assignmentVariants = ref<Variant[]>([])
const submissions = ref<Submission[]>([])
const activeSub = ref<Submission|null>(null)
const loadingSubs = ref(false)
const grading = ref(false)
const bulkGrading = ref(false)
const bulkDone = ref(0)
const bulkTotal = ref(0)
const submitting = ref(false)
const uploading = ref(false)
const retracting = ref(false)
const fileInputEl = ref<HTMLInputElement>()
const uploadedUrl = ref('')
const submittedFiles = ref<File[]>([])
const uploadIdxSub = ref(0)
const uploadPctSub = ref(0)
const form = ref({ text: '', file: null as File|null, variantNumber: null as number|null })

const filteredSubmissions = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  if (!q) return submissions.value
  return submissions.value.filter(s => {
    const name = (studentMap.value[s.student_id] || '').toLowerCase()
    return name.includes(q)
  })
})

const parsedCriteria = computed(() => { try { return JSON.parse(props.assignment.criteria) } catch { return [] } })
const parsedCriteriaScores = computed(() => { if (!mySubmission.value?.grade?.criteria_scores) return null; try { return JSON.parse(mySubmission.value.grade.criteria_scores) } catch { return null } })
const parsedActiveScores = computed(() => { if (!activeSub.value?.grade?.criteria_scores) return null; try { return JSON.parse(activeSub.value.grade.criteria_scores) } catch { return null } })

const parseFileUrls = (raw?: string | null): string[] => {
  if (!raw) return []
  try { const arr = JSON.parse(raw); return Array.isArray(arr) ? arr : [] } catch { return [] }
}
const parsedSubmittedUrls = computed(() => parseFileUrls(mySubmission.value?.file_urls))
const parsedActiveUrls = computed(() => parseFileUrls(activeSub.value?.file_urls))

const assignmentFiles = computed(() => {
  const desc = props.assignment.description || ''
  const urlRe = /(https?:\/\/[^\s\n"'<>]+)/g
  const extRe = /\.(pdf|doc|docx|txt|md|ppt|pptx|xls|xlsx|png|jpg|jpeg|gif|webp)(\?[^\s]*)?$/i
  const files: {name: string; url: string}[] = []
  for (const m of [...desc.matchAll(urlRe)]) {
    const url = m[1].replace(/[.,;:!?)]+$/, '')
    if (extRe.test(url)) {
      files.push({ url, name: getFileName(url) })
    }
  }
  return files
})

const parseUtc = (d: string) => new Date(d.endsWith('Z') || d.includes('+') ? d : d + 'Z')
const parsedDeadline = computed(() => props.assignment.deadline ? parseUtc(props.assignment.deadline) : null)
const isOverdue = computed(() => parsedDeadline.value ? parsedDeadline.value < new Date() : false)
const deadlineStr = computed(() => parsedDeadline.value?.toLocaleString('ru-RU', { day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit' }) ?? '')
const canSubmit = computed(() =>
  (form.value.text.trim() || submittedFiles.value.length) &&
  !submitting.value &&
  (assignmentVariants.value.length === 0 || form.value.variantNumber !== null)
)

const getStudentName = (id: number) => studentMap.value[id] || `Студент #${id}`
const getStudentInitials = (id: number) => { const fn = studentMap.value[id]; if (!fn) return String(id); const parts = fn.trim().split(' ').filter(Boolean); return parts.map((p:string) => p[0]).join('').toUpperCase().slice(0, 2) || String(id) }
const statusLabel = (s: string) => ({ submitted: 'Выполнено', grading: 'Проверяется', graded: 'Оценено', late: 'Просрочено' }[s] || s)
const fmtDate = (d: string) => parseUtc(d).toLocaleString('ru-RU', { day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit' })
const getFileName = (url: string) => { try { return decodeURIComponent(new URL(url).pathname.split('/').pop() || url) } catch { return url.slice(-40) } }
const getEmoji = (url: string) => {
  const e = url.split('.').pop()?.split('?')[0]?.toLowerCase() || ''
  if (e === 'pdf') return '📄'
  if (['doc','docx'].includes(e)) return '📝'
  if (['txt','md'].includes(e)) return '📃'
  if (['xls','xlsx'].includes(e)) return '📊'
  if (['ppt','pptx'].includes(e)) return '📋'
  if (['png','jpg','jpeg','gif','webp'].includes(e)) return '🖼️'
  return '📎'
}
const fileSz = (f: File) => f.size < 1048576 ? (f.size/1024).toFixed(0)+' KB' : (f.size/1048576).toFixed(1)+' MB'

const clearFiles = () => { submittedFiles.value = []; uploadedUrl.value = '' }
const clearFile = () => { submittedFiles.value = []; uploadedUrl.value = '' }
const onDrop = (e: DragEvent) => {
  const files = Array.from(e.dataTransfer?.files || [])
  submittedFiles.value = [...submittedFiles.value, ...files]
}
const onFileSelect = (e: Event) => {
  const files = Array.from((e.target as HTMLInputElement).files || [])
  submittedFiles.value = [...submittedFiles.value, ...files]
  ;(e.target as HTMLInputElement).value = ''
}

const loadSubs = async () => {
  if (loadingSubs.value) return
  searchQuery.value = ''
  loadingSubs.value = true
  try {
    const [subs, users] = await Promise.all([
      svc.getSubmissions(props.assignment.id),
      usersSvc.all()
    ])
    submissions.value = subs
    const fnReg: Record<number, string> = JSON.parse(localStorage.getItem('_fullname_registry') || '{}')
    const nickReg: Record<number, string> = JSON.parse(localStorage.getItem('_nick_registry') || '{}')
    const map: Record<number, string> = {}
    for (const u of users) {
      map[u.id] = u.full_name || u.fullName || fnReg[u.id] || nickReg[u.id] || u.name || u.email || `Студент #${u.id}`
    }
    for (const sub of subs) {
      if (sub.student_name && sub.student_id) {
        map[sub.student_id] = sub.student_name
      }
    }
    studentMap.value = map
  }
  catch { toast.err('Ошибка загрузки работ') }
  finally { loadingSubs.value = false }
}

const runAiGrade = async () => {
  if (!activeSub.value || grading.value) return
  grading.value = true
  try {
    const grade = await svc.aiGrade(activeSub.value.id)
    activeSub.value = { ...activeSub.value, grade, status: 'graded' }
    const idx = submissions.value.findIndex(s => s.id === activeSub.value!.id)
    if (idx !== -1) submissions.value[idx] = { ...submissions.value[idx], grade, status: 'graded' }
    toast.ok(`ИИ проверил: ${grade.score} / ${props.assignment.max_score}`)
  } catch (e: any) { toast.err(e?.response?.data?.detail || 'Ошибка ИИ-проверки') }
  finally { grading.value = false }
}

const runBulkAiGrade = async () => {
  if (bulkGrading.value) return
  const pending = submissions.value.filter(s => s.status === 'submitted' || s.status === 'late')
  if (!pending.length) return
  bulkGrading.value = true
  bulkDone.value = 0
  bulkTotal.value = pending.length
  let ok = 0
  for (const sub of pending) {
    try {
      const grade = await svc.aiGrade(sub.id)
      const idx = submissions.value.findIndex(s => s.id === sub.id)
      if (idx !== -1) submissions.value[idx] = { ...submissions.value[idx], grade, status: 'graded' }
      ok++
    } catch {}
    bulkDone.value++
  }
  bulkGrading.value = false
  toast.ok(`ИИ проверил ${ok} из ${pending.length} работ`)
}

const retract = async () => {
  if (!mySubmission.value || retracting.value) return
  retracting.value = true
  try {
    await svc.retractSubmission(mySubmission.value.id)
    mySubmission.value = null
    form.value = { text: '', file: null }
    uploadedUrl.value = ''
    toast.ok('Сдача отозвана — можно отправить заново')
    emit('submitted', null)
  } catch (e: any) { toast.err(e?.response?.data?.detail || 'Ошибка отзыва сдачи') }
  finally { retracting.value = false }
}

const doSubmit = async () => {
  if (!canSubmit.value || submitting.value) return
  submitting.value = true
  try {
    const fileUrls: string[] = []
    if (submittedFiles.value.length) {
      uploading.value = true
      for (let i = 0; i < submittedFiles.value.length; i++) {
        uploadIdxSub.value = i + 1
        uploadPctSub.value = Math.round(((i + 1) / submittedFiles.value.length) * 100)
        const res = await uploadSvc.upload(submittedFiles.value[i])
        fileUrls.push(res.file_url)
      }
      uploading.value = false
    }
    const sub = await svc.submit(props.assignment.id, {
      text_content: form.value.text.trim() || undefined,
      file_urls: fileUrls.length ? fileUrls : undefined,
      variant_number: form.value.variantNumber ?? undefined,
      student_name: auth.fullname || undefined,
    })
    mySubmission.value = sub
    toast.ok('Работа успешно сдана!')
    emit('submitted', sub)
  } catch (e: any) { toast.err(e?.response?.data?.detail || 'Ошибка при сдаче') }
  finally { submitting.value = false; uploading.value = false }
}

onMounted(async () => {
  try {
    assignmentVariants.value = await svc.listVariants(props.assignment.id)
  } catch {}

  if (!canSeeSubmissions.value) {
    try {
      const subs = await svc.mySubmissions()
      mySubmission.value = subs.find(s => s.assignment_id === props.assignment.id) ?? null
      if (mySubmission.value) tab.value = 'submit'
    } catch {}
  }
})
</script>

<style scoped src="~/assets/components/assignmentModal.css"></style>
