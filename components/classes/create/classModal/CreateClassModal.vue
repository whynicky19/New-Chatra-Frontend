<template>
  <div class="overlay" @click.self="$emit('close')">
    <div class="modal anim-scale" style="max-width:480px;width:100%">
      <div class="modal-head">
        <span class="modal-title">Создать класс</span>
        <button class="btn btn-icon btn-ghost" @click="$emit('close')">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
        </button>
      </div>

      <!-- Cover image upload -->
      <div class="frow">
        <label class="flabel">Обложка класса</label>
        <div class="cover-upload" @click="fileInput?.click()" :style="coverPreview ? `background-image:url(${coverPreview})` : ''">
          <input ref="fileInput" type="file" accept="image/*" style="display:none" @change="onImagePick"/>
          <div v-if="!coverPreview" class="cover-placeholder">
            <div class="cover-ph-icon">
              <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="18" height="18" rx="3"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21 15 16 10 5 21"/></svg>
            </div>
            <span>Нажмите для загрузки фото</span>
            <span class="cover-ph-hint">JPG, PNG — будет обложкой карточки</span>
          </div>
          <button v-else class="cover-remove" @click.stop="coverPreview=null;coverBase64=null">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
          </button>
        </div>
      </div>

      <div class="frow"><label class="flabel">Название класса *</label><input v-model="title" class="input" placeholder="Например: Математика 10А" autofocus/></div>
      <div class="frow"><label class="flabel">Описание</label><input v-model="description" class="input" placeholder="Краткое описание курса"/></div>
      <div class="frow"><label class="flabel">Период</label><input v-model="period" class="input" placeholder="Например: 2024-2025"/></div>
      <div class="frow"><label class="flabel">Учитель / Преподаватель</label><input v-model="teacher" class="input" placeholder="Ваше имя"/></div>

      <!-- Группа -->
      <div class="frow" style="position:relative">
        <label class="flabel">Группа</label>
        <input
          v-model="groupQuery"
          class="input"
          placeholder="Например: ИСУ-21"
          autocomplete="off"
          @input="onGroupInput"
          @blur="onGroupBlur"
          @focus="showGroupDropdown = true"
        />
        <div v-if="showGroupDropdown && groupSuggestions.length" class="group-dropdown">
          <div
            v-for="g in groupSuggestions"
            :key="g"
            class="group-item"
            @mousedown.prevent="selectGroup(g)"
          >
            {{ g }}
          </div>
        </div>
        <div v-if="group" class="nick-hint ok">✓ {{ group }}</div>
      </div>

      <!-- Общие критерии оценивания -->
      <div class="frow">
        <div class="criteria-header">
          <label class="flabel">Общие критерии оценивания</label>
          <button class="btn-add-crit" type="button" @click="addCriterion">+ Добавить</button>
        </div>
        <div class="criteria-hint">ИИ будет использовать их для всех заданий без собственных критериев</div>
        <div class="criteria-list">
          <div v-for="(c, i) in defaultCriteria" :key="i" class="crit-row">
            <span class="crit-num">{{ i + 1 }}</span>
            <input v-model="c.name" class="input input-sm" placeholder="Название критерия" />
            <input v-model.number="c.weight" type="number" class="input input-sm weight-inp" min="1" placeholder="Баллы" />
            <button class="crit-rm" type="button" @click="defaultCriteria.splice(i,1)">×</button>
          </div>
          <div v-if="!defaultCriteria.length" class="no-crit">Критерии не заданы — ИИ оценит по умолчанию</div>
        </div>
      </div>

      <div class="modal-foot">
        <button class="btn btn-white" @click="$emit('close')">Отмена</button>
        <button class="btn btn-blue" :disabled="!title.trim()||loading" @click="submit">
          <div v-if="loading" class="spinner" style="width:13px;height:13px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
          <span v-else>Создать класс</span>
        </button>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import { useToast } from '~/composables/useToast/useToast'
import { usePostsSvc } from '~/services/posts'
import { useApi } from '~/services/api'
const emit = defineEmits<{close:[]; created:[p:any]}>()
const postsSvc = usePostsSvc(); const toast = useToast(); const api = useApi()
const title = ref(''); const period = ref(''); const teacher = ref(''); const description = ref(''); const loading = ref(false)
const coverPreview = ref<string|null>(null); const coverBase64 = ref<string|null>(null)
const fileInput = ref<HTMLInputElement|null>(null)
const defaultCriteria = ref<{name: string; weight: number}[]>([])

// группа
const group = ref('')
const groupQuery = ref('')
const groupSuggestions = ref<string[]>([])
const showGroupDropdown = ref(false)

const onGroupInput = async () => {
  group.value = ''
  const q = groupQuery.value.trim()
  if (!q) { groupSuggestions.value = []; showGroupDropdown.value = false; return }
  try {
    const { data } = await api.get(`/auth/groups/search?q=${encodeURIComponent(q)}`)
    groupSuggestions.value = data
    showGroupDropdown.value = groupSuggestions.value.length > 0
  } catch {
    groupSuggestions.value = []
  }
}

const selectGroup = (g: string) => {
  group.value = g
  groupQuery.value = g
  showGroupDropdown.value = false
}

const onGroupBlur = () => {
  setTimeout(() => { showGroupDropdown.value = false }, 150)
}

const addCriterion = () => defaultCriteria.value.push({ name: '', weight: 10 })

const onImagePick = (e: Event) => {
  const f = (e.target as HTMLInputElement).files?.[0]; if (!f) return
  const reader = new FileReader()
  reader.onload = (ev) => {
    const result = ev.target?.result as string
    coverPreview.value = result; coverBase64.value = result
  }
  reader.readAsDataURL(f)
}

const submit = async () => {
  loading.value = true
  try {
    const body = JSON.stringify({
      type: 'class', period: period.value, teacher: teacher.value,
      description: description.value, members: 1, color: '',
      cover_image: coverBase64.value || '',
      group: group.value || '',
      default_criteria: defaultCriteria.value.filter(c => c.name.trim() && c.weight > 0)
    })
    const p = await postsSvc.create(title.value, body)
    toast.ok('Класс создан'); emit('created', p)
  } catch(e: any) { toast.err(e?.response?.data?.detail || 'Ошибка') }
  finally { loading.value = false }
}
</script>

<style scoped src="~/assets/components/createClassModal.css"></style>

