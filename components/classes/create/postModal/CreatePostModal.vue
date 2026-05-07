<template>
  <div class="overlay" @click.self="$emit('close')">
    <div class="modal anim-scale">
      <div class="modal-head">
        <div class="modal-head-l">
          <div class="modal-ico" :class="type === 'lecture' ? 'ico-lec' : 'ico-mat'">
            <svg v-if="type === 'lecture'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 3h6a4 4 0 014 4v14a3 3 0 00-3-3H2z"/><path d="M22 3h-6a4 4 0 00-4 4v14a3 3 0 013-3h7z"/></svg>
            <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
          </div>
          <div>
            <div class="modal-title">{{ type === 'lecture' ? 'Добавить лекцию' : 'Добавить материал' }}</div>
            <div class="modal-sub">{{ type === 'lecture' ? 'Учебный материал для класса' : 'Файлы и ресурсы для студентов' }}</div>
          </div>
        </div>
        <button class="btn btn-icon btn-ghost" @click="$emit('close')">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
        </button>
      </div>

      <!-- Type switch -->
      <div class="type-switch">
        <button :class="['type-btn', { active: type === 'lecture' }]" @click="type = 'lecture'">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 3h6a4 4 0 014 4v14a3 3 0 00-3-3H2z"/><path d="M22 3h-6a4 4 0 00-4 4v14a3 3 0 013-3h7z"/></svg>
          Лекция
        </button>
        <button :class="['type-btn', { active: type === 'hw' }]" @click="type = 'hw'">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
          Материал
        </button>
      </div>

      <div class="modal-body">
        <div class="field">
          <label class="field-label">{{ type === 'lecture' ? 'Тема лекции' : 'Название материала' }} *</label>
          <input v-model="title" class="inp" :placeholder="type === 'lecture' ? 'Например: Введение в тему...' : 'Название книги, ресурса...'" autofocus />
        </div>
        <div class="field">
          <label class="field-label">{{ type === 'lecture' ? 'Содержание лекции' : 'Описание' }}</label>
          <textarea v-model="body" class="inp inp-ta" rows="4" :placeholder="type === 'lecture' ? 'Текст лекции, ссылки на видео...' : 'Краткое описание материала...'"></textarea>
        </div>

        <!-- File upload -->
        <div class="field">
          <label class="field-label">Прикрепить файлы</label>
          <div class="file-drop" :class="{ dragging: drag }" @dragover.prevent="drag=true" @dragleave="drag=false" @drop.prevent="onDrop" @click="fi?.click()">
            <input type="file" style="display:none" ref="fi" multiple accept="image/*,.pdf,.doc,.docx,.xls,.xlsx,.txt,.zip,.ppt,.pptx" @change="onPick" />
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" class="drop-icon"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
            <span class="drop-text">Перетащите или <strong>нажмите для выбора</strong></span>
            <span class="drop-hint">PDF, DOCX, PPT, изображения и другие</span>
          </div>

          <div v-if="selFiles.length" class="files-list">
            <div v-for="(f, i) in selFiles" :key="i" class="file-item">
              <span class="file-emoji">{{ fileIcon(f) }}</span>
              <span class="file-name">{{ f.name }}</span>
              <span class="file-size">{{ fileSize(f) }}</span>
              <button class="file-rm" @click.stop="removeFile(i)">
                <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
              </button>
            </div>
          </div>
        </div>

        <!-- Upload progress -->
        <div v-if="uploading" class="upload-progress">
          <div class="upload-bar-wrap">
            <div class="upload-bar" :style="{ width: uploadPct + '%' }"></div>
          </div>
          <span class="upload-text">Загрузка {{ uploadCurrent }}/{{ selFiles.length }}...</span>
        </div>
      </div>

      <div class="modal-foot">
        <button class="btn btn-ghost" @click="$emit('close')">Отмена</button>
        <button class="btn btn-purple" :disabled="!title.trim() || loading" @click="submit">
          <div v-if="loading" class="spinner"></div>
          <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12h14"/></svg>
          {{ type === 'lecture' ? 'Опубликовать лекцию' : 'Добавить материал' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useToast } from '~/composables/useToast/useToast'
import { usePostsSvc } from '~/services/posts'
import { useUploadSvc } from '~/services/uploads'

const props = defineProps<{ classId: number }>()
const emit = defineEmits<{ close: []; created: [p: any] }>()

const postsSvc = usePostsSvc()
const uploadSvc = useUploadSvc()
const toast = useToast()

const type = ref<'lecture' | 'hw'>('lecture')
const title = ref('')
const body = ref('')
const loading = ref(false)
const selFiles = ref<File[]>([])
const drag = ref(false)
const fi = ref<HTMLInputElement | null>(null)
const uploading = ref(false)
const uploadCurrent = ref(0)
const uploadPct = ref(0)

const fileIcon = (f: File) => {
  if (f.type.startsWith('image/')) return '🖼️'
  const ext = f.name.split('.').pop()?.toLowerCase() || ''
  return ({ pdf: '📄', doc: '📝', docx: '📝', xls: '📊', xlsx: '📊', ppt: '📋', pptx: '📋', zip: '🗜️', txt: '📃' })[ext] || '📎'
}
const fileSize = (f: File) => f.size < 1024 * 1024 ? (f.size / 1024).toFixed(0) + ' KB' : (f.size / 1024 / 1024).toFixed(1) + ' MB'

const onPick = (e: Event) => {
  selFiles.value = [...selFiles.value, ...Array.from((e.target as HTMLInputElement).files || [])]
}
const onDrop = (e: DragEvent) => {
  drag.value = false
  selFiles.value = [...selFiles.value, ...Array.from(e.dataTransfer?.files || [])]
}
const removeFile = (i: number) => { selFiles.value = selFiles.value.filter((_, idx) => idx !== i) }

const submit = async () => {
  loading.value = true
  uploading.value = selFiles.value.length > 0
  uploadCurrent.value = 0
  uploadPct.value = 0
  try {
    let finalBody = body.value
    const urls: string[] = []
    for (let i = 0; i < selFiles.value.length; i++) {
      uploadCurrent.value = i + 1
      uploadPct.value = Math.round(((i + 1) / selFiles.value.length) * 100)
      const { file_url } = await uploadSvc.upload(selFiles.value[i])
      urls.push(`${file_url}`)
    }
    if (urls.length > 0) finalBody += '\n\n' + urls.join('\n')

    const p = await postsSvc.create(
      `[${type.value.toUpperCase()}][${props.classId}] ${title.value}`,
      finalBody
    )
    toast.ok(type.value === 'lecture' ? 'Лекция опубликована' : 'Материал добавлен')
    emit('created', p)
  } catch (e: any) {
    toast.err(e?.response?.data?.detail || 'Ошибка')
  } finally {
    loading.value = false
    uploading.value = false
  }
}
</script>

<style scoped src="~/assets/components/createPostModal.css"></style>
