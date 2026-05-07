<template>
  <div :class="['cli',{active}]" @click="$emit('click')">
    <div class="cli-av-wrap">
      <img v-if="chatAvatar" :src="chatAvatar" class="cli-av-img"/>
      <div v-else :class="['cli-av-init', gradFor(chat.id)]">{{chat.name[0]?.toUpperCase()||'#'}}</div>
    </div>
    <div style="flex:1;min-width:0">
      <div class="truncate cli-name">{{chat.name}}</div>
      <div class="cli-sub truncate">{{lastMsg || 'Нет сообщений'}}</div>
    </div>
    <div v-if="unreadCount" class="cli-badge">{{unreadCount > 99 ? '99+' : unreadCount}}</div>
  </div>
</template>
<script setup lang="ts">
import { computed } from 'vue'
import { useChatsStore } from '~/stores/chats.store'
import { useAuthStore } from '~/stores/auth.store'
import type { Chat } from '~/stores/chats.store'
const props = defineProps<{chat:Chat;active:boolean}>()
defineEmits<{click:[]}>()
const chatsStore = useChatsStore()
const auth = useAuthStore()
const grads=['g0','g1','g2','g3','g4','g5']
const gradFor=(id:number)=>grads[id%grads.length]
const avatarReg=()=>{try{return JSON.parse(localStorage.getItem('_avatar_registry')||'{}')}catch{return{}}}
const chatAvatar=computed(()=>{
  const users=chatsStore.chatUsers[props.chat.id]||[]
  const other=users.find((u:any)=>u.id!==auth.user?.id)
  if(!other) return ''
  const reg=avatarReg()
  if(other.id===auth.user?.id&&auth.avatar) return auth.avatar
  return reg[other.id]||''
})
const lastMsg=computed(()=>{
  const msgs=chatsStore.messages[props.chat.id]
  if(!msgs?.length) return ''
  const m=msgs[msgs.length-1]
  if(m?.content?.startsWith('🖼️')) return '📷 Фото'
  if(m?.content?.startsWith('📎')) return '📎 Файл'
  return m?.content?.slice(0,40)||''
})
const unreadCount=computed(()=>chatsStore.unread[props.chat.id]||0)
</script>

<style scoped src="~/assets/components/chatListItem.css"></style>