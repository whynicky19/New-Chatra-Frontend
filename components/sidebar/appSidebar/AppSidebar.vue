<template>
  <aside :class="['sb', { collapsed: isCollapsed }]">
    <!-- Logo — click to toggle sidebar -->
    <div class="sb-logo" @click="toggleSidebar">
      <!-- Expanded: new logo icon + Chatra text -->
      <template v-if="!isCollapsed">
        <img src="/logo-icon.png" class="logo-img-new" alt="Chatra"/>
        <span class="logo-name">Chatra</span>
      </template>
      <!-- Collapsed: old small logo -->
      <template v-else>
        <img src="/logo.png" class="logo-img-collapsed" alt="Chatra"/>
      </template>
    </div>

    <!-- Language switcher -->
    <div class="lang-switch" v-if="!isCollapsed">
      <button :class="['lang-btn', { active: lang === 'ru' }]" @click.stop="setLang('ru')">RU</button>
      <button :class="['lang-btn', { active: lang === 'en' }]" @click.stop="setLang('en')">EN</button>
      <button :class="['lang-btn', { active: lang === 'kk' }]" @click.stop="setLang('kk')">KZ</button>
    </div>

    <nav class="sb-nav">
      <NuxtLink to="/" class="sb-item" :class="{active:route.path==='/'||route.path.startsWith('/classes')}">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M2 3h6a4 4 0 014 4v14a3 3 0 00-3-3H2z"/><path d="M22 3h-6a4 4 0 00-4 4v14a3 3 0 013-3h7z"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.classes') }}</span>
      </NuxtLink>

      <NuxtLink to="/chats" class="sb-item" :class="{active:route.path==='/chats'||route.path.startsWith('/chats')}">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/></svg>
          <span v-if="totalUnread>0" class="notif-dot">{{totalUnread>9?'9+':totalUnread}}</span>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.chats') }}</span>
        <span v-if="!isCollapsed && !isMobile && totalUnread>0" class="notif-pill">{{totalUnread>99?'99+':totalUnread}}</span>
      </NuxtLink>

      <NuxtLink v-if="auth.isAdmin" to="/admin" class="sb-item" :class="{active:route.path==='/admin'}">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.participants') }}</span>
      </NuxtLink>

      <NuxtLink to="/ai" class="sb-item" :class="{active:route.path==='/ai'}">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.ai') }}</span>
      </NuxtLink>

      <NuxtLink to="/settings" class="sb-item" :class="{active:route.path==='/settings'}">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 010 2.83 2 2 0 01-2.83 0l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 01-4 0v-.09A1.65 1.65 0 009 19.4a1.65 1.65 0 00-1.82.33l-.06.06a2 2 0 01-2.83-2.83l.06-.06A1.65 1.65 0 004.68 15a1.65 1.65 0 00-1.51-1H3a2 2 0 010-4h.09A1.65 1.65 0 004.6 9a1.65 1.65 0 00-.33-1.82l-.06-.06a2 2 0 012.83-2.83l.06.06A1.65 1.65 0 009 4.68a1.65 1.65 0 001-1.51V3a2 2 0 014 0v.09a1.65 1.65 0 001 1.51 1.65 1.65 0 001.82-.33l.06-.06a2 2 0 012.83 2.83l-.06.06A1.65 1.65 0 0019.4 9a1.65 1.65 0 001.51 1H21a2 2 0 010 4h-.09a1.65 1.65 0 00-1.51 1z"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.settings') }}</span>
      </NuxtLink>
    </nav>

    <!-- Nudge: нет ФИО -->
    <div v-if="!isCollapsed && !isMobile && !auth.fullname" class="fio-nudge" @click="$router.push('/settings')">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
      <span>Укажите ваше ФИО в настройках</span>
    </div>

    <div class="sb-bottom">
      <!-- Mobile language switcher (hidden on desktop — shown in page header) -->
      <div class="lang-switch-mobile">
        <button v-for="l in [{code:'ru',label:'RU'},{code:'en',label:'EN'},{code:'kk',label:'KZ'}]" :key="l.code"
          :class="['lang-btn-mob', { active: lang === l.code }]" @click.stop="setLang(l.code as any)">{{ l.label }}</button>
      </div>
      <a href="https://t.me/whynicky" target="_blank" class="sb-item help-item" :title="t('chats.help_center')">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="12" r="10"/><path d="M9.09 9a3 3 0 015.83 1c0 2-3 3-3 3"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('chats.help_center') }}</span>
      </a>
      <div class="sb-item logout-item" @click="doLogout" :title="t('nav.logout')">
        <div class="item-icon">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M9 21H5a2 2 0 01-2-2V5a2 2 0 012-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
        </div>
        <span class="item-label" v-if="!isCollapsed && !isMobile">{{ t('nav.logout') }}</span>
      </div>
    </div>
  </aside>
</template>
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute } from '#app'
import { useAuthStore } from '~/stores/auth.store'
import { useChatsStore } from '~/stores/chats.store'
import { useAuth } from '~/composables/useAuth/useAuth'
import { useI18n } from '~/composables/usel18n/useI18n'
import { useAi, AI_LIMIT } from '~/composables/useAi/useAi'
const auth = useAuthStore(); const chatsStore = useChatsStore(); const { logout } = useAuth(); const route = useRoute()
const { t, lang, setLang } = useI18n()
const ai = useAi()
const showAiQuota = computed(() => auth.user?.role === 'student' && !auth.user?.ai_unlimited)
const aiRemaining = computed(() => ai.aiRemaining.value)
const aiLimitReached = computed(() => ai.aiLimitReached.value)
const aiQuotaColor = computed(() => {
  if (aiLimitReached.value) return 'red'
  if (aiRemaining.value <= 2) return 'yellow'
  return 'teal'
})
const totalUnread = computed(() => chatsStore.totalUnread)
const doLogout = () => { chatsStore.disconnectAll(); logout() }

const isCollapsed = ref(false)
const isMobile = ref(false)
const toggleSidebar = () => {
  isCollapsed.value = !isCollapsed.value
  if (import.meta.client) localStorage.setItem('_sidebar_collapsed', isCollapsed.value ? '1' : '0')
}
onMounted(() => {
  if (import.meta.client) {
    isCollapsed.value = localStorage.getItem('_sidebar_collapsed') === '1'
    const check = () => { isMobile.value = window.innerWidth <= 768 }
    check()
    window.addEventListener('resize', check)
  }
})
</script>

<style scoped src="~/assets/components/appSidebar.css"></style>