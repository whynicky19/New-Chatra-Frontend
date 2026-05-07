<template>
  <div :class="['chats-layout', isMobile && chatsStore.active ? 'mobile-chat' : isMobile ? 'mobile-list' : '']">
    <!-- Chat list panel -->
    <div class="chat-list-panel">
      <div class="clp-head">
        <h2 class="clp-title">{{ t('nav.chats') }}</h2>
        <button class="btn btn-icon btn-ghost" @click="showNew=true">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M5 12h14"/></svg>
        </button>
      </div>
      <div class="clp-search">
        <div class="search-wrap">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--text4);flex-shrink:0"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
          <input v-model="searchQ" class="search-inp" :placeholder="t('chats.search')" @input="onSearch" @keydown.escape="clearSearch"/>
          <button v-if="searchQ" class="search-clear" @click="clearSearch">×</button>
        </div>
      </div>

      <Transition name="slide">
        <div v-if="searchQ" class="sr-panel">
          <div v-if="sLoading" class="sr-loading"><div class="spinner" style="width:16px;height:16px;border-width:2px"></div></div>
          <template v-else>
            <template v-if="sResults.length">
              <div class="sr-head">{{ t('chats.users_found') }} · {{sResults.length}}</div>
              <div v-for="u in sResults" :key="u.id" class="sr-item" @click="openDM(u)">
                <div class="item-av">
                  <img v-if="getAvatar(u.id)" :src="getAvatar(u.id)" class="av-img"/>
                  <div v-else :class="['av','av-sm','bg-b0']">{{displayInit(u)}}</div>
                </div>
                <div style="flex:1;min-width:0">
                  <div class="truncate" style="font-size:13px;font-weight:600;color:var(--text1)">{{displayName(u)}}</div>
                  <div style="font-size:11px;color:var(--text4)">{{u.email}}</div>
                </div>
                <div class="sr-btn"><svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/></svg></div>
              </div>
            </template>
            <div v-else class="sr-empty">{{ t('chats.no_users') }}</div>
          </template>
        </div>
      </Transition>

      <div v-show="!searchQ" class="chats-scroll">
        <div v-if="chatsStore.loadingChats" class="list-loading"><div class="spinner"></div></div>
        <div v-else-if="!chatsStore.chats.length" class="empty" style="padding:28px 16px">
          <div style="font-size:36px;margin-bottom:8px">💬</div>
          <div class="empty-title">{{ t('chats.no_chats') }}</div>
          <div class="empty-sub">{{ t('chats.no_chats_sub') }}</div>
        </div>
        <TransitionGroup v-else name="chat-list" tag="div">
          <div v-for="c in chatsStore.chats" :key="c.id" :class="['chat-item',{active:chatsStore.active?.id===c.id}]" @click="selectChat(c)">
            <div class="item-av">
              <img v-if="getChatAvatar(c)" :src="getChatAvatar(c)" class="av-img"/>
              <div v-else :class="['av','av-sm','bg-b'+((c.id)%6)]">{{chatInit(c)}}</div>
            </div>
            <div style="flex:1;min-width:0">
              <div class="truncate chat-name">{{chatTitle(c)}}</div>
              <div class="truncate chat-preview">{{lastPreview(c.id)}}</div>
            </div>
            <div class="chat-meta">
              <div class="chat-time">{{ chatTime(c.id) }}</div>
              <div v-if="unread(c.id)" class="unread-badge">{{unread(c.id)>99?'99+':unread(c.id)}}</div>
            </div>
          </div>
        </TransitionGroup>
      </div>
    </div>

    <!-- Main chat area -->
    <div class="chat-main">
      <Transition name="chat-switch" mode="out-in">
        <ChatWindow v-if="chatsStore.active" :key="chatsStore.active.id" :show-back="isMobile" @back="mobileBack"/>
        <div v-else class="no-chat" key="empty">
          <div class="no-chat-icon">💬</div>
          <div class="no-chat-title">{{ t('chats.select') }}</div>
          <div class="no-chat-sub">{{ t('chats.select_sub') }}</div>
        </div>
      </Transition>
    </div>

    <!-- New chat modal -->
    <div v-if="showNew" class="overlay" @click.self="showNew=false">
      <div class="modal anim-scale">
        <div class="modal-head">
          <span class="modal-title">{{ t('chats.new_group') }}</span>
          <button class="btn btn-icon btn-ghost" @click="showNew=false"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg></button>
        </div>
        <div class="frow"><label class="flabel">{{ t('general.name') }}</label><input v-model="newName" class="input" :placeholder="t('chats.group_name')" autofocus @keyup.enter="createGroup"/></div>
        <div class="modal-foot">
          <button class="btn btn-white" @click="showNew=false">{{ t('general.cancel') }}</button>
          <button class="btn btn-teal" :disabled="!newName.trim()||creating" @click="createGroup">
            <div v-if="creating" class="spinner" style="width:13px;height:13px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
            <span v-else>{{ t('general.save') }}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { useAuthStore } from '~/stores/auth.store'
import { useChatsStore } from '~/stores/chats.store'
import { useToast } from '~/composables/useToast/useToast'
import { useI18n } from '~/composables/usel18n/useI18n'
import { useChatsSvc } from '~/services/chats'
import { useUsersSvc } from '~/services/users'
import { useChat } from '~/composables/useChat/useChat'
definePageMeta({ layout: 'default' })
const auth=useAuthStore();const chatsStore=useChatsStore();const chatsSvc=useChatsSvc();const usersSvc=useUsersSvc();const toast=useToast();const{connectWs,loadUsers,loadMsgs,startChatPoller}=useChat()
const { t } = useI18n()
const searchQ=ref('');const sResults=ref<any[]>([]);const sLoading=ref(false);const showNew=ref(false);const newName=ref('');const creating=ref(false)
let timer:any=null
const isMobile=ref(false)
const mobileBack=()=>{chatsStore.setActive(null)}
let _resizeHandler:()=>void
const nickReg=():Record<string,string>=>{try{return JSON.parse(localStorage.getItem('_nick_registry')||'{}')}catch{return{}}}
const avatarReg=():Record<string,string>=>{try{return JSON.parse(localStorage.getItem('_avatar_registry')||'{}')}catch{return{}}}
const getAvatar=(uid:number):string=>uid===auth.user?.id&&auth.avatar?auth.avatar:(avatarReg()[uid]||'')
const displayName=(u:any):string=>nickReg()[u.id]||u.full_name||u.email.split('@')[0]
const displayInit=(u:any):string=>displayName(u)[0]?.toUpperCase()||'?'
const chatTitle=(c:any):string=>{const users=chatsStore.chatUsers[c.id]||[];const other=users.find((u:any)=>u.id!==auth.user?.id);if(other){const n=nickReg()[other.id];return n||other.email.split('@')[0]};return c.name?.startsWith('Чат с ')?c.name.replace('Чат с ',''):c.name||''}
const chatInit=(c:any):string=>chatTitle(c)[0]?.toUpperCase()||'#'
const getChatAvatar=(c:any):string=>{const users=chatsStore.chatUsers[c.id]||[];const other=users.find((u:any)=>u.id!==auth.user?.id);if(other)return getAvatar(other.id);return''}
const lastPreview=(id:number):string=>{const m=chatsStore.messages[id];if(!m?.length)return t('chats.no_messages');const last=m[m.length-1];if(last.content?.startsWith('🖼️'))return'📷 Фото';if(last.content?.startsWith('📎'))return'📎 Файл';return last.content?.slice(0,45)||''}
const chatTime=(id:number):string=>{const m=chatsStore.messages[id];if(!m?.length)return'';const last=m[m.length-1];if(!last?.created_at)return'';try{const d=new Date(last.created_at);const now=new Date();const diff=now.getTime()-d.getTime();if(diff<86400000)return d.toLocaleTimeString([],{hour:'2-digit',minute:'2-digit'});return d.toLocaleDateString([],{month:'short',day:'numeric'})}catch{return''}}
const unread=(id:number)=>chatsStore.unread[id]||0
const onSearch=()=>{clearTimeout(timer);sResults.value=[];const q=searchQ.value.trim().toLowerCase();if(!q)return;sLoading.value=true;timer=setTimeout(async()=>{try{const all=await usersSvc.all();sResults.value=all.filter((u:any)=>u.id!==auth.user?.id&&(u.email.toLowerCase().includes(q)||(u.full_name||'').toLowerCase().includes(q)))}catch{sResults.value=[]}finally{sLoading.value=false}},300)}
const clearSearch=()=>{searchQ.value='';sResults.value=[]}
const openDM=async(user:any)=>{const exist=chatsStore.chats.find(c=>{const us=chatsStore.chatUsers[c.id]||[];return us.length===2&&us.some((u:any)=>u.id===user.id)&&us.some((u:any)=>u.id===auth.user?.id)});if(exist){chatsStore.setActive(exist);chatsStore.markRead(exist.id);clearSearch();return};try{const c=await chatsSvc.create(`Чат с ${user.email}`);await chatsSvc.addUser(c.id,user.id);connectWs(c.id);await loadUsers(c.id);chatsStore.addChat(c);chatsStore.setActive(c);clearSearch();toast.ok(t('chats.created'))}catch{toast.err(t('chats.error'))}}
const selectChat=(c:any)=>{chatsStore.setActive(c);chatsStore.markRead(c.id)}
const createGroup=async()=>{if(!newName.value.trim())return;creating.value=true;try{const c=await chatsSvc.create(newName.value.trim());chatsStore.addChat(c);chatsStore.setActive(c);connectWs(c.id);showNew.value=false;newName.value='';toast.ok(t('chats.created'))}catch{toast.err(t('chats.error'))}finally{creating.value=false}}
onMounted(async()=>{
  _resizeHandler=()=>{isMobile.value=window.innerWidth<=768}
  _resizeHandler()
  window.addEventListener('resize',_resizeHandler)
  if(!chatsStore.chats.length){chatsStore.loadingChats=true;try{const c=await chatsSvc.list();chatsStore.setChats(c);await Promise.all(c.map(async(ch:any)=>{connectWs(ch.id);await loadUsers(ch.id)}));await Promise.all(c.map((ch:any)=>loadMsgs(ch.id)))}catch{toast.err(t('chats.error'))}finally{chatsStore.loadingChats=false}}else{chatsStore.chats.forEach((ch:any)=>{if(!chatsStore.chatUsers[ch.id]?.length)loadUsers(ch.id);if(!chatsStore.messages[ch.id]?.length)loadMsgs(ch.id)})};startChatPoller()
})
onUnmounted(()=>{if(import.meta.client)window.removeEventListener('resize',_resizeHandler)})
</script>

<style scoped src="~/assets/pages/chats.css"></style>
