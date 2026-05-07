<template>
  <div class="cw">
    <!-- Header -->
    <div class="cw-head">
      <div class="cw-head-l">
        <button v-if="props.showBack" class="btn btn-icon btn-ghost back-btn" @click="emit('back')">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M19 12H5M12 5l-7 7 7 7"/></svg>
        </button>
        <div class="cw-av">
          <img v-if="chatAvatar" :src="chatAvatar" class="cw-av-img"/>
          <div v-else :class="['av','av-sm',colorFor(chatsStore.active?.id||0)]">{{cInit}}</div>
        </div>
        <div>
          <div class="cw-name">{{chatTitle}}</div>
          <div class="cw-sub">
            <span :class="wsOk?'st-dot-on':'st-dot-off'"></span>
            <span :class="wsOk?'st-on':'st-off'">{{wsOk?(lang==='ru'?'В сети':'Online'):(lang==='ru'?'Оффлайн':'Offline')}}</span>
            <span class="cw-role">· {{ chatRole }}</span>
          </div>
        </div>
      </div>
      <div class="cw-head-r">
        <button class="btn btn-icon btn-ghost" @click="showSearch=!showSearch" title="Search">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
        </button>
        <button class="btn btn-icon btn-ghost" @click="showMembers=!showMembers" title="Участники">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg>
        </button>
        <button class="btn btn-icon btn-ghost" @click="showAddModal=true" title="Добавить участника">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M16 11c1.66 0 2.99-1.34 2.99-3S17.66 5 16 5c-1.66 0-3 1.34-3 3s1.34 3 3 3zm-8 0c1.66 0 2.99-1.34 2.99-3S9.66 5 8 5C6.34 5 5 6.34 5 8s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V19h14v-2.5c0-2.33-4.67-3.5-7-3.5zm8 0c-.29 0-.62.02-.97.05 1.16.84 1.97 1.97 1.97 3.45V19h6v-2.5c0-2.33-4.67-3.5-7-3.5z"/></svg>
        </button>
      </div>
    </div>

    <!-- Search bar -->
    <Transition name="slide-down">
      <div v-if="showSearch" class="search-bar">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--text4);flex-shrink:0"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
        <input v-model="searchQ" class="search-inp" placeholder="Поиск в сообщениях..." @keydown.escape="showSearch=false;searchQ=''"/>
      </div>
    </Transition>

    <!-- Members panel -->
    <Transition name="slide-down">
      <div v-if="showMembers&&chatsStore.activeUsers.length" class="members-bar">
        <div v-for="u in chatsStore.activeUsers" :key="u.id" class="member-chip">
          <img v-if="getAvatar(u.id)" :src="getAvatar(u.id)" class="chip-av-img"/>
          <div v-else :class="['av','av-xs',colorFor(u.id)]">{{getNickInit(u)}}</div>
          <span class="truncate" style="font-size:12px;font-weight:500;max-width:110px">{{getNick(u)}}</span>
          <button v-if="u.id!==auth.user?.id" class="chip-remove" @click="removeUser(u.id)" title="Удалить">×</button>
        </div>
      </div>
    </Transition>

    <!-- Messages -->
    <div ref="area" class="msg-area">
      <div v-if="chatsStore.loadingMsgs" class="empty">
        <div class="spinner" style="width:24px;height:24px;border-width:3px"></div>
      </div>
      <div v-else-if="!filteredMsgs.length" class="empty">
        <div class="empty-icon">💬</div>
        <div class="empty-title">{{searchQ?'Ничего не найдено':'Нет сообщений'}}</div>
        <div class="empty-sub">{{searchQ?'Попробуйте другой запрос':'Начните переписку'}}</div>
      </div>
      <div v-else class="msgs">
        <div style="height:12px"></div>
        <ChatMessage
          v-for="(msg,i) in filteredMsgs"
          :key="msg.id"
          :message="msg"
          :is-own="msg.user_id===auth.user?.id"
          :chat-users="chatsStore.activeUsers"
          :show-name="shouldShowName(filteredMsgs,i)"
          @delete="deleteMsg(msg.id)"
          @scroll="scrollBottom"
        />
        <div style="height:12px"></div>
      </div>
    </div>

    <!-- Input -->
    <div class="inp-wrap">
      <div v-if="pendingFile" class="file-preview">
        <span>{{pendingFile.type.startsWith('image/')?'🖼️':'📎'}} {{pendingFile.name}}</span>
        <button class="btn btn-icon btn-ghost btn-sm" @click="pendingFile=null">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
        </button>
      </div>
      <div class="inp-row">
        <button class="emoji-btn" title="Emoji">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="12" r="10"/><path d="M8 14s1.5 2 4 2 4-2 4-2"/><line x1="9" y1="9" x2="9.01" y2="9"/><line x1="15" y1="9" x2="15.01" y2="9"/></svg>
        </button>
        <textarea
          ref="ta"
          v-model="msgText"
          class="msg-inp"
          :placeholder="lang==='ru'?'Введите ваш запрос здесь...':'Type your message here...'"
          rows="1"
          @keydown.enter.exact.prevent="sendMessage"
          @input="resizeTA"
        ></textarea>
        <label class="attach-btn" title="Прикрепить файл">
          <input type="file" accept="image/*,.pdf,.doc,.docx,.xls,.xlsx,.txt,.zip,.rar" style="display:none" @change="onFilePick"/>
          <div class="attach-icon">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M21.44 11.05l-9.19 9.19a6 6 0 01-8.49-8.49l9.19-9.19a4 4 0 015.66 5.66l-9.2 9.19a2 2 0 01-2.83-2.83l8.49-8.48"/></svg>
          </div>
        </label>
        <button
          :class="['send-btn',{active:msgText.trim()||pendingFile}]"
          :disabled="(!msgText.trim()&&!pendingFile)||sending"
          @click="sendMessage"
        >
          <div v-if="sending" class="spinner" style="width:14px;height:14px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
          <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/></svg>
        </button>
      </div>
    </div>

    <!-- Add member modal -->
    <div v-if="showAddModal" class="overlay" @click.self="closeAdd">
      <div class="modal anim-scale">
        <div class="modal-head">
          <span class="modal-title">Добавить участника</span>
          <button class="btn btn-icon btn-ghost" @click="closeAdd">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg>
          </button>
        </div>
        <div class="frow">
          <label class="flabel">Поиск по @нику</label>
          <div class="add-search">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--text4);flex-shrink:0"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
            <input v-model="addQ" class="add-inp" placeholder="@никнейм..." @input="onAddSearch"/>
          </div>
        </div>
        <div v-if="addLoading" style="display:flex;justify-content:center;padding:12px"><div class="spinner"></div></div>
        <div v-if="addResults.length" class="add-results">
          <div v-for="u in addResults" :key="u.id" class="add-row">
            <img v-if="getAvatar(u.id)" :src="getAvatar(u.id)" class="av-img-xs"/>
            <div v-else :class="['av','av-sm',colorFor(u.id)]">{{getNickInit(u)}}</div>
            <div style="flex:1;min-width:0">
              <div class="truncate" style="font-size:13px;font-weight:500;color:var(--text1)">{{getNick(u)}}</div>
              <div style="font-size:11px;color:var(--text4)">{{u.email}}</div>
            </div>
            <button
              class="btn btn-blue btn-sm"
              :disabled="isMember(u.id)||addingId===u.id"
              @click="doAddUser(u)"
            >
              <div v-if="addingId===u.id" class="spinner" style="width:12px;height:12px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
              <span v-else-if="isMember(u.id)">В чате</span>
              <span v-else>Добавить</span>
            </button>
          </div>
        </div>
        <!-- Current members -->
        <div v-if="chatsStore.activeUsers.length" style="margin-top:16px">
          <div class="flabel">Участники ({{chatsStore.activeUsers.length}})</div>
          <div class="members-list">
            <div v-for="u in chatsStore.activeUsers" :key="u.id" class="member-row">
              <img v-if="getAvatar(u.id)" :src="getAvatar(u.id)" class="av-img-xs"/>
              <div v-else :class="['av','av-xs',colorFor(u.id)]">{{getNickInit(u)}}</div>
              <span class="truncate" style="font-size:13px;flex:1;color:var(--text1)">{{getNick(u)}}</span>
              <button v-if="u.id!==auth.user?.id" class="btn btn-danger btn-sm" @click="removeUser(u.id)">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue'
import { useAuthStore } from '~/stores/auth.store'
import { useChatsStore } from '~/stores/chats.store'
import { useChat } from '~/composables/useChat/useChat'
import { useToast } from '~/composables/useToast/useToast'
import { useUploadSvc } from '~/services/uploads'
import { useChatsSvc } from '~/services/chats'
import { useUsersSvc } from '~/services/users'
import { useI18n } from '~/composables/usel18n/useI18n'

const props = defineProps<{ showBack?: boolean }>()
const emit = defineEmits<{ (e: 'back'): void }>()

const auth = useAuthStore()
const chatsStore = useChatsStore()
const { lang } = useI18n()
const { loadMsgs, loadUsers, connectWs, sendMsg, delMsg } = useChat()

const otherUserId = computed(() => {
  const users = chatsStore.activeUsers
  return users.find(u => u.id !== auth.user?.id)?.id ?? null
})

const wsOk = computed(() => {
  const uid = otherUserId.value
  if (!uid) return false
  const msgs = chatsStore.activeMsgs
  const last = [...msgs].reverse().find(m => m.user_id === uid)
  if (!last?.created_at) return false
  return Date.now() - new Date(last.created_at).getTime() < 10 * 60 * 1000
})
const uploadSvc = useUploadSvc()
const chatsSvc = useChatsSvc()
const usersSvc = useUsersSvc()
const toast = useToast()

const area = ref<HTMLElement|null>(null)
const ta = ref<HTMLTextAreaElement|null>(null)
const msgText = ref('')
const sending = ref(false)
const pendingFile = ref<File|null>(null)
const showSearch = ref(false)
const showMembers = ref(false)
const showAddModal = ref(false)
const searchQ = ref('')

const addQ = ref('')
const addResults = ref<any[]>([])
const addLoading = ref(false)
const addingId = ref<number|null>(null)
let addTimer: any = null

const avColors = ['bg-b0','bg-b1','bg-b2','bg-b3','bg-b4','bg-b5']
const colorFor = (id: number) => avColors[id % avColors.length]

const nickReg = () => { try { return JSON.parse(localStorage.getItem('_nick_registry')||'{}') } catch { return {} } }
const avatarReg = () => { try { return JSON.parse(localStorage.getItem('_avatar_registry')||'{}') } catch { return {} } }
const getNick = (u: any): string => u.full_name || nickReg()[u.id] || u.email.split('@')[0]
const getNickInit = (u: any): string => getNick(u)[0]?.toUpperCase()||'?'
const getAvatar = (uid: number): string => uid===auth.user?.id&&auth.avatar ? auth.avatar : (avatarReg()[uid]||'')

const chatTitle = computed(() => {
  const users = chatsStore.activeUsers
  const other = users.find(u => u.id !== auth.user?.id)
  if (other && users.length <= 2) {
    if (other.full_name) return other.full_name
    const nick = nickReg()[other.id]
    if (nick) return nick
    return other.email.split('@')[0]
  }
  return chatsStore.active?.name || ''
})

const chatAvatar = computed(() => {
  const users = chatsStore.activeUsers
  const other = users.find(u => u.id !== auth.user?.id)
  if (other && users.length <= 2) return getAvatar(other.id)
  return ''
})

const cInit = computed(() => chatTitle.value.replace('@','')[0]?.toUpperCase()||'#')

const chatRole = computed(() => {
  const users = chatsStore.activeUsers
  const other = users.find(u => u.id !== auth.user?.id)
  if (!other) return ''
  if (other.role === 'admin') return lang.value === 'ru' ? 'Администратор' : 'Administrator'
  if (other.role === 'teacher') return lang.value === 'ru' ? 'Научный руководитель' : 'Supervisor'
  return lang.value === 'ru' ? 'Студент' : 'Student'
})

const filteredMsgs = computed(() => {
  if (!searchQ.value) return chatsStore.activeMsgs
  return chatsStore.activeMsgs.filter(m => m.content?.toLowerCase().includes(searchQ.value.toLowerCase()))
})

const shouldShowName = (msgs: any[], i: number) => {
  if (msgs[i].user_id === auth.user?.id) return false
  return !msgs[i-1] || msgs[i-1].user_id !== msgs[i].user_id
}

const isMember = (uid: number) => chatsStore.activeUsers.some(u => u.id === uid)

const scrollBottom = () => nextTick(() => { if (area.value) area.value.scrollTop = area.value.scrollHeight })
const resizeTA = () => { if (!ta.value) return; ta.value.style.height='auto'; ta.value.style.height=Math.min(ta.value.scrollHeight,140)+'px' }
const onFilePick = (e: Event) => { const f=(e.target as HTMLInputElement).files?.[0]; if(f) pendingFile.value=f }

const closeAdd = () => { showAddModal.value=false; addQ.value=''; addResults.value=[] }

const onAddSearch = () => {
  clearTimeout(addTimer)
  addResults.value = []
  const q = addQ.value.trim().replace('@','').toLowerCase()
  if (!q) return
  addLoading.value = true
  addTimer = setTimeout(async () => {
    try {
      const all = await usersSvc.all()
      const reg = nickReg()
      addResults.value = all.filter((u: any) => {
        const nick = (reg[u.id]||'').toLowerCase()
        return nick.includes(q) && u.id !== auth.user?.id
      })
    } catch { addResults.value=[] }
    finally { addLoading.value=false }
  }, 300)
}

const doAddUser = async (user: any) => {
  if (!chatsStore.active) return
  addingId.value = user.id
  try {
    await chatsSvc.addUser(chatsStore.active.id, user.id)
    await loadUsers(chatsStore.active.id)
    toast.ok(`${getNick(user)} добавлен`)
  } catch (e: any) { toast.err(e?.response?.data?.detail||'Ошибка') }
  finally { addingId.value=null }
}

const removeUser = async (uid: number) => {
  if (!chatsStore.active) return
  try {
    await chatsSvc.removeUser(chatsStore.active.id, uid)
    await loadUsers(chatsStore.active.id)
    toast.ok('Участник удалён')
  } catch { toast.err('Ошибка') }
}

const sendMessage = async () => {
  if (!chatsStore.active || (!msgText.value.trim() && !pendingFile.value)) return
  sending.value = true
  try {
    let content = msgText.value.trim()
    if (pendingFile.value) {
      const { file_url } = await uploadSvc.upload(pendingFile.value)
      const isImg = pendingFile.value.type.startsWith('image/')
      content = isImg
        ? `🖼️ [Фото](${file_url}) — ${pendingFile.value.name}`
        : `📎 [Файл](${file_url}) — ${pendingFile.value.name}`
      pendingFile.value = null
    }
    if (content) {
      const ok = await sendMsg(chatsStore.active.id, content)
      if (ok) { msgText.value=''; if(ta.value) ta.value.style.height='auto' }
    }
    scrollBottom()
  } finally { sending.value=false }
}

const deleteMsg = (id: number) => { if (chatsStore.active) delMsg(chatsStore.active.id, id) }

watch(() => chatsStore.active, async c => {
  if (c) {
    searchQ.value = ''
    await Promise.all([loadMsgs(c.id), loadUsers(c.id)])
    connectWs(c.id)
    scrollBottom()
  }
}, { immediate: true })

watch(() => chatsStore.activeMsgs.length, () => scrollBottom())
</script>

<style scoped src="~/assets/components/chatWindow.css"></style>