<template>
  <div class="overlay" @click.self="$emit('close')">
    <div class="modal anim-scale">
      <div class="modal-head">
        <div class="modal-head-l">
          <div class="modal-ico">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M11 4H4a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 013 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>
          </div>
          <div>
            <div class="modal-title">Новое задание</div>
            <div class="modal-sub">Заполните данные задания</div>
          </div>
        </div>
        <button class="btn btn-icon btn-ghost" @click="$emit('close')">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
        </button>
      </div>

      <div class="modal-body">
        <div class="field">
          <label class="field-label">Название задания *</label>
          <input v-model="form.title" class="inp" placeholder="Например: Контрольная работа по теме..." />
        </div>

        <div class="field">
          <label class="field-label">Описание задания</label>
          <textarea v-model="form.description" class="inp inp-ta" rows="3" placeholder="Подробное описание, требования, инструкции..."></textarea>
        </div>

        <!-- Файлы задания -->
        <div class="field">
          <label class="field-label">Прикрепить файлы к заданию</label>
          <div class="file-drop" :class="{ dragging: drag }" @dragover.prevent="drag=true" @dragleave="drag=false" @drop.prevent="onDropFiles" @click="taskFileInput?.click()">
            <input type="file" style="display:none" ref="taskFileInput" multiple @change="onPickFiles" />
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" style="color:var(--text4)"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
            <span style="font-size:13px;color:var(--text3)">Перетащите или <strong style="color:var(--teal)">выберите файлы</strong> задания</span>
          </div>
          <div v-if="taskFiles.length" class="attached-files">
            <div v-for="(f, i) in taskFiles" :key="i" class="attached-file">
              <span>{{ fileEmoji(f) }}</span>
              <span class="af-name">{{ f.name }}</span>
              <span class="af-size">{{ fileSz(f) }}</span>
              <button class="af-rm" @click="taskFiles.splice(i,1)">×</button>
            </div>
          </div>
          <div v-if="uploadingTask" class="upload-prog">
            <div class="upload-bar" :style="{ width: uploadPct + '%' }"></div>
            <span>Загрузка {{ uploadIdx }}/{{ taskFiles.length }}...</span>
          </div>
        </div>

        <!-- ─── ЭТАЛОННЫЕ РЕШЕНИЯ (несколько файлов) ────────────────────── -->
        <div class="field ref-section">
          <div class="ref-header">
            <div class="ref-header-l">
              <div class="ref-ico">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                  <path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 01-2 2H5a2 2 0 01-2-2V5a2 2 0 012-2h11"/>
                </svg>
              </div>
              <div>
                <div class="ref-title">Эталонные решения</div>
                <div class="ref-desc">ИИ сравнит работы учеников со всеми прикреплёнными файлами</div>
              </div>
            </div>
            <div class="ref-badge">ИИ</div>
          </div>

          <!-- Список прикреплённых эталонов -->
          <div v-if="refFiles.length" class="attached-files" style="margin-bottom:8px">
            <div v-for="(rf, i) in refFiles" :key="i" class="attached-file ref-attached ref-done">
              <span>{{ fileEmoji(rf.file) }}</span>
              <div class="ref-file-info">
                <span class="af-name" style="color:var(--green)">{{ rf.name }}</span>
                <span class="ref-hint">{{ rf.url ? 'Загружен' : 'Будет загружен' }}</span>
              </div>
              <span class="af-size">{{ rf.file ? fileSz(rf.file) : '' }}</span>
              <button class="af-rm" @click="refFiles.splice(i,1)">×</button>
            </div>
          </div>

          <!-- Зона загрузки (всегда видна — можно добавлять ещё) -->
          <div class="file-drop file-drop-ref" :class="{ dragging: dragRef }"
               @dragover.prevent="dragRef=true" @dragleave="dragRef=false"
               @drop.prevent="onDropRef" @click="refFileInput?.click()">
            <input type="file" style="display:none" ref="refFileInput" multiple @change="onPickRef" />
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" style="color:var(--green)"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
            <span style="font-size:13px;color:var(--text3)">
              {{ refFiles.length ? 'Добавить ещё файл' : 'Перетащите или' }}
              <strong v-if="!refFiles.length" style="color:var(--green)">выберите файлы</strong>
              <strong v-else style="color:var(--green)">эталонного решения</strong>
            </span>
            <span class="ref-formats">PDF, DOCX, DOC, PPTX, XLSX, TXT, MD... · Несколько файлов</span>
          </div>

          <!-- Прогресс загрузки -->
          <div v-if="uploadingRef" class="upload-prog">
            <div class="upload-bar upload-bar-green"></div>
            <span style="color:var(--green)">Загрузка эталонных решений...</span>
          </div>
        </div>
        <!-- ─────────────────────────────────────────────────────────────── -->

        <div class="field-row">
          <div class="field">
            <label class="field-label">Макс. балл</label>
            <input v-model.number="form.max_score" type="number" class="inp" min="1" max="1000" />
          </div>
          <div class="field">
            <label class="field-label">Дедлайн</label>
            <input v-model="form.deadline" type="datetime-local" class="inp" />
          </div>
        </div>

        <!-- Criteria -->
        <div class="field">
          <div class="criteria-head">
            <label class="field-label">Критерии оценивания</label>
            <button class="btn btn-ghost btn-sm" @click="addCriterion">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
              Добавить
            </button>
          </div>
          <div class="criteria-hint">Сумма весов критериев должна быть равна макс. баллу ({{ totalWeight }}/{{ form.max_score }})</div>

          <div class="criteria-list">
            <div v-for="(c, i) in form.criteria" :key="i" class="criterion-row">
              <div class="criterion-num">{{ i + 1 }}</div>
              <div class="criterion-fields">
                <input v-model="c.name" class="inp inp-sm" placeholder="Название критерия" />
                <input v-model="c.description" class="inp inp-sm" placeholder="Описание (необязательно)" />
              </div>
              <div class="criterion-weight">
                <input v-model.number="c.weight" type="number" class="inp inp-sm weight-inp" min="1" placeholder="Баллы" />
              </div>
              <button class="btn btn-icon btn-ghost btn-sm del-btn" @click="removeCriterion(i)">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
              </button>
            </div>

            <div v-if="!form.criteria.length" class="no-criteria">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" opacity="0.4"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 01-2 2H5a2 2 0 01-2-2V5a2 2 0 012-2h11"/></svg>
              <span>Добавьте критерии оценивания</span>
            </div>
          </div>
        </div>

        <!-- ─── ВАРИАНТЫ ─────────────────────────────────────────────── -->
        <div class="field var-section">
          <div class="ref-header">
            <div class="ref-header-l">
              <div class="ref-ico" style="background:rgba(0,177,201,.1);border-color:rgba(0,177,201,.25);color:var(--teal)">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M8 6h13M8 12h13M8 18h13M3 6h.01M3 12h.01M3 18h.01"/></svg>
              </div>
              <div>
                <div class="ref-title">Варианты заданий</div>
                <div class="ref-desc">Добавьте варианты с отдельными эталонными решениями (необязательно)</div>
              </div>
            </div>
            <button class="btn btn-ghost btn-sm" @click="addVariant">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
              Вариант
            </button>
          </div>
          <div v-for="(v, i) in variants" :key="i" class="variant-row">
            <div class="variant-num">{{ v.number }}</div>
            <input v-model="v.title" class="inp inp-sm" placeholder="Название (необязательно)" style="flex:1" />
            <div v-if="!v.fileUrl && !v.file" class="var-upload-btn" @click="triggerVarFile(i)">
              <input type="file" style="display:none" :ref="el => { if(el) varFileInputs[i] = el as HTMLInputElement }" @change="onPickVarFile($event, i)" />
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
              Эталон
            </div>
            <div v-else class="var-file-chip">
              <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" style="color:var(--green)"><polyline points="20 6 9 17 4 12"/></svg>
              <span>{{ v.file ? v.file.name.slice(0,20) : 'Загружен' }}</span>
              <button class="af-rm" @click="v.file=null;v.fileUrl=''">×</button>
            </div>
            <button class="btn btn-icon btn-ghost btn-sm del-btn" @click="variants.splice(i,1)">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
            </button>
          </div>
          <div v-if="!variants.length" class="no-criteria" style="padding:12px">
            <span>Без вариантов — одно общее эталонное решение</span>
          </div>
        </div>
        <!-- ──────────────────────────────────────────────────────────── -->

      </div>

      <div class="modal-foot">
        <button class="btn btn-ghost" @click="$emit('close')">Отмена</button>
        <button class="btn btn-purple" :disabled="!canSubmit || saving" @click="submit">
          <div v-if="saving" class="spinner" style="width:14px;height:14px;border-width:2px"></div>
          <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
          Создать задание
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useAssignmentsSvc } from '~/services/assignments'
import { useUploadSvc } from '~/services/uploads'
import { useToast } from '~/composables/useToast/useToast'

const emit = defineEmits(['close', 'created'])
const props = defineProps<{ classId: number }>()
const svc = useAssignmentsSvc()
const uploadSvc = useUploadSvc()
const toast = useToast()
const saving = ref(false)
const drag = ref(false)
const dragRef = ref(false)
const taskFileInput = ref<HTMLInputElement>()
const refFileInput = ref<HTMLInputElement>()
const taskFiles = ref<File[]>([])

// Multi-file reference solutions
interface RefFileDraft { name: string; file: File | null; url: string }
const refFiles = ref<RefFileDraft[]>([])

const uploadingTask = ref(false)
const uploadingRef = ref(false)
const uploadIdx = ref(0)
const uploadPct = ref(0)

// ─── Variants ────────────────────────────────────────────────────────────────
interface VariantDraft { number: number; title: string; file: File | null; fileUrl: string }
const variants = ref<VariantDraft[]>([])
const varFileInputs = ref<HTMLInputElement[]>([])
const addVariant = () => variants.value.push({ number: variants.value.length + 1, title: '', file: null, fileUrl: '' })
const triggerVarFile = (i: number) => varFileInputs.value[i]?.click()
const onPickVarFile = (e: Event, i: number) => {
  const f = (e.target as HTMLInputElement).files?.[0]
  if (f) { variants.value[i].file = f; variants.value[i].fileUrl = '' }
}

const fileEmoji = (f: File | null) => {
  if (!f) return '📎'
  const e = f.name.split('.').pop()?.toLowerCase() || ''
  if (e === 'pdf') return '📄'
  if (['doc','docx'].includes(e)) return '📝'
  if (['txt','md'].includes(e)) return '📃'
  if (['xls','xlsx'].includes(e)) return '📊'
  if (['ppt','pptx'].includes(e)) return '📋'
  if (['png','jpg','jpeg','gif','webp'].includes(e)) return '🖼️'
  return '📎'
}
const fileSz = (f: File | null) => {
  if (!f) return ''
  return f.size < 1048576 ? (f.size/1024).toFixed(0)+' KB' : (f.size/1048576).toFixed(1)+' MB'
}

const onPickFiles = (e: Event) => {
  taskFiles.value = [...taskFiles.value, ...Array.from((e.target as HTMLInputElement).files || [])]
}
const onDropFiles = (e: DragEvent) => {
  drag.value = false
  taskFiles.value = [...taskFiles.value, ...Array.from(e.dataTransfer?.files || [])]
}
const onPickRef = (e: Event) => {
  const files = Array.from((e.target as HTMLInputElement).files || [])
  for (const f of files) {
    refFiles.value.push({ name: f.name, file: f, url: '' })
  }
  // reset input so same file can be re-added if needed
  if (refFileInput.value) refFileInput.value.value = ''
}
const onDropRef = (e: DragEvent) => {
  dragRef.value = false
  const files = Array.from(e.dataTransfer?.files || [])
  for (const f of files) {
    refFiles.value.push({ name: f.name, file: f, url: '' })
  }
}

const form = ref({
  title: '',
  description: '',
  max_score: 100,
  deadline: '',
  criteria: [{ name: '', weight: 100, description: '' }]
})

const totalWeight = computed(() => form.value.criteria.reduce((s, c) => s + (c.weight || 0), 0))
const canSubmit = computed(() =>
  form.value.title.trim() &&
  form.value.criteria.length > 0 &&
  form.value.criteria.every(c => c.name.trim() && c.weight > 0)
)

const addCriterion = () => form.value.criteria.push({ name: '', weight: 0, description: '' })
const removeCriterion = (i: number) => form.value.criteria.splice(i, 1)

const submit = async () => {
  if (!canSubmit.value || saving.value) return
  saving.value = true
  try {
    let description = form.value.description.trim() || undefined

    // Загружаем файлы задания
    if (taskFiles.value.length) {
      uploadingTask.value = true
      const urls: string[] = []
      for (let i = 0; i < taskFiles.value.length; i++) {
        uploadIdx.value = i + 1
        uploadPct.value = Math.round(((i + 1) / taskFiles.value.length) * 100)
        const { file_url } = await uploadSvc.upload(taskFiles.value[i])
        urls.push(file_url)
      }
      uploadingTask.value = false
      if (urls.length) {
        description = (description ? description + '\n\n' : '') + urls.join('\n')
      }
    }

    // Загружаем эталонные решения (несколько файлов)
    const resolvedRefUrls: string[] = []
    if (refFiles.value.length) {
      uploadingRef.value = true
      for (const rf of refFiles.value) {
        if (rf.url) {
          resolvedRefUrls.push(rf.url)
        } else if (rf.file) {
          const { file_url } = await uploadSvc.upload(rf.file)
          resolvedRefUrls.push(file_url)
        }
      }
      uploadingRef.value = false
    }

    // Store multiple URLs as JSON array (backend supports both single string and JSON array)
    const resolvedRefUrl = resolvedRefUrls.length === 1
      ? resolvedRefUrls[0]
      : resolvedRefUrls.length > 1
        ? JSON.stringify(resolvedRefUrls)
        : undefined

    const body: any = {
      class_id: props.classId,
      title: form.value.title.trim(),
      description,
      criteria: form.value.criteria.map(c => ({ name: c.name, weight: c.weight, description: c.description || undefined })),
      max_score: form.value.max_score,
      reference_solution_url: resolvedRefUrl,
    }
    if (form.value.deadline) body.deadline = new Date(form.value.deadline).toISOString()

    const created = await svc.create(body)

    // Загружаем варианты с эталонами
    for (const v of variants.value) {
      let url = v.fileUrl
      if (v.file) {
        const { file_url } = await uploadSvc.upload(v.file)
        url = file_url
      }
      if (url) {
        await svc.addVariant(created.id, { variant_number: v.number, title: v.title || undefined, reference_solution_url: url })
      }
    }

    toast.ok('Задание создано')
    emit('created', created)
  } catch (e: any) {
    toast.err(e?.response?.data?.detail || 'Ошибка создания задания')
  } finally {
    saving.value = false
    uploadingTask.value = false
    uploadingRef.value = false
  }
}
</script>

<style scoped src="~/assets/components/createAssignmentModal.css"></style>