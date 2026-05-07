<template>
  <div class="pg">
    <div class="pg-head">
      <div>
        <h1 class="pg-title">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          Админ-панель
        </h1>
      </div>
    </div>
    <div class="pg-body">
      <!-- Stats -->
      <div class="stats-row">
        <div class="stat-card">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="1.8"><path d="M17 21v-2a4 4 0 00-4-4H5a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 00-3-3.87M16 3.13a4 4 0 010 7.75"/></svg>
          <div class="stat-val">{{users.length}}</div>
          <div class="stat-lbl">Пользователей</div>
        </div>
        <div class="stat-card">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="1.8"><path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/></svg>
          <div class="stat-val">{{chatsCount}}</div>
          <div class="stat-lbl">Чатов</div>
        </div>
        <div class="stat-card">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="1.8"><path d="M22 10v6M2 10l10-5 10 5-10 5z"/><path d="M6 12v5c3 3 9 3 12 0v-5"/></svg>
          <div class="stat-val">{{classesCount}}</div>
          <div class="stat-lbl">Классов</div>
        </div>
      </div>

      <!-- Tabs -->
      <div class="tabs" style="margin-bottom:20px">
        <button :class="['tab-btn',{active:tab==='users'}]" @click="tab='users'">Пользователи</button>
        <button :class="['tab-btn',{active:tab==='actions'}]" @click="tab='actions'">Действия</button>
        <button :class="['tab-btn','tab-ai-btn',{active:tab==='ai-usage'}]" @click="switchToAiUsage">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
          Запросы к ИИ
        </button>
      </div>

      <!-- Users tab -->
      <div v-if="tab==='users'">
        <div style="display:flex;gap:8px;margin-bottom:12px;align-items:center">
          <div class="search-wrap" style="flex:1">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--text4)"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
            <input v-model="sq" class="search-inp" placeholder="Поиск пользователей..."/>
          </div>
          <button class="btn btn-blue btn-sm" @click="showCreate=true">+ Создать</button>
        </div>
        <div v-if="loadingU" style="display:flex;justify-content:center;padding:24px"><div class="spinner"></div></div>
        <div v-else class="users-table">
          <table>
            <thead><tr><th>Имя</th><th>Email</th><th>Роль</th><th>ИИ доступ</th><th>Дата регистрации</th><th></th></tr></thead>
            <tbody>
              <tr v-for="u in fUsers" :key="u.id">
                <td>
                  <div style="display:flex;align-items:center;gap:8px">
                    <div :class="['av','av-sm',colorFor(u.id)]">{{(u.full_name || u.email)[0].toUpperCase()}}</div>
                    <span style="font-size:13px;font-weight:500">{{u.full_name || u.email.split('@')[0]}}</span>
                  </div>
                </td>
                <td style="font-size:13px;color:var(--text3)">{{u.email}}</td>
                <td>
                  <span :class="['badge',u.role==='admin'?'badge-blue':'badge-gray']">{{u.role}}</span>
                </td>
                <td>
                  <div v-if="u.role==='student'" style="display:flex;align-items:center;gap:6px">
                    <span :class="['ai-quota-badge', u.ai_unlimited ? 'unlimited' : 'limited']">
                      <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
                      {{ u.ai_unlimited ? 'Безлимит' : '5 запросов' }}
                    </span>
                    <button
                      :class="['btn btn-sm', u.ai_unlimited ? 'btn-ghost' : 'btn-blue']"
                      :disabled="togglingAi[u.id]"
                      style="font-size:11px;padding:3px 8px"
                      @click="toggleAiUnlimited(u)"
                    >
                      <div v-if="togglingAi[u.id]" class="spinner" style="width:10px;height:10px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div>
                      <span v-else>{{ u.ai_unlimited ? 'Ограничить' : 'Разрешить' }}</span>
                    </button>
                  </div>
                  <span v-else style="font-size:12px;color:var(--text4)">—</span>
                </td>
                <td style="font-size:12px;color:var(--text4)">{{today}}</td>
                <td>
                  <div style="display:flex;gap:4px">
                    <select :value="u.role" class="role-sel" :disabled="u.id===auth.user?.id" @change="setRole(u.id,($event.target as HTMLSelectElement).value)">
                      <option value="student">student</option>
                      <option value="teacher">teacher</option>
                      <option value="admin">admin</option>
                    </select>
                    <button v-if="u.is_active" class="btn btn-icon btn-ghost btn-sm" :disabled="u.id===auth.user?.id" @click="doBlock(u.id)" title="Заблокировать">
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="4.93" y1="4.93" x2="19.07" y2="19.07"/></svg>
                    </button>
                    <button v-else class="btn btn-icon btn-ghost btn-sm" @click="doUnblock(u.id)" title="Разблокировать">
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="20 6 9 17 4 12"/></svg>
                    </button>
                    <button class="btn btn-icon btn-danger btn-sm" :disabled="u.id===auth.user?.id" @click="doDel(u.id)">
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a1 1 0 011-1h4a1 1 0 011 1v2"/></svg>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="!fUsers.length"><td colspan="6" style="text-align:center;padding:24px;color:var(--text4);font-size:13px">Пользователи не найдены</td></tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Actions tab -->
      <div v-else-if="tab==='actions'">
        <div class="action-cards">
          <div class="action-card">
            <div class="action-icon" style="background:var(--blue-l)">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="2"><path d="M16 21v-2a4 4 0 00-4-4H6a4 4 0 00-4 4v2"/><circle cx="9" cy="7" r="4"/><line x1="19" y1="8" x2="19" y2="14"/><line x1="22" y1="11" x2="16" y2="11"/></svg>
            </div>
            <div>
              <div class="action-title">Создать пользователя</div>
              <div class="action-desc">Добавить нового участника</div>
            </div>
            <button class="btn btn-blue btn-sm" @click="showCreate=true">Создать</button>
          </div>
        </div>
      </div>

      <!-- AI Usage tab -->
      <div v-else-if="tab==='ai-usage'">
        <!-- Summary cards per class -->
        <div v-if="aiSummary.length" class="ai-summary-row">
          <div
            v-for="s in aiSummary"
            :key="s.class_id ?? 'general'"
            :class="['ai-sum-card', { active: aiFilterClass === s.class_id }]"
            @click="setClassFilter(s.class_id)"
          >
            <div class="ai-sum-top">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
              <span class="ai-sum-label">{{ s.class_id ? 'Класс #' + s.class_id : 'Общий чат' }}</span>
            </div>
            <div class="ai-sum-tokens">{{ s.total_tokens.toLocaleString() }}</div>
            <div class="ai-sum-meta">токенов · {{ s.request_count }} запросов</div>
          </div>
        </div>

        <!-- Controls bar -->
        <div class="ai-controls">
          <div class="ai-filter-row">
            <div class="search-wrap" style="flex:1;max-width:260px">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--text4)"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
              <select v-model="aiFilterClass" class="search-inp" style="cursor:pointer" @change="loadAiUsage(1)">
                <option :value="null">Все классы</option>
                <option v-for="s in aiSummary" :key="s.class_id ?? 'g'" :value="s.class_id">
                  {{ s.class_id ? 'Класс #' + s.class_id : 'Общий чат' }}
                </option>
              </select>
            </div>
            <div class="ai-total-badge" v-if="aiTotal > 0">
              Итого: {{ aiTotalTokens.toLocaleString() }} токенов
            </div>
            <button class="btn btn-ghost btn-sm" @click="loadAiUsage(aiPage)" :disabled="aiLoading">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" :class="aiLoading?'spin':''"><path d="M23 4v6h-6M1 20v-6h6"/><path d="M3.51 9a9 9 0 0114.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0020.49 15"/></svg>
              Обновить
            </button>
          </div>
        </div>

        <!-- Table -->
        <div v-if="aiLoading" style="display:flex;justify-content:center;padding:32px"><div class="spinner"></div></div>
        <div v-else class="users-table ai-table">
          <table>
            <thead>
              <tr>
                <th style="width:50px">#</th>
                <th>Дата</th>
                <th>Класс</th>
                <th>Тип</th>
                <th>Пользователь</th>
                <th style="text-align:right">Входящие</th>
                <th style="text-align:right">Исходящие</th>
                <th style="text-align:right">Всего токенов</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, idx) in aiLogs" :key="item.id">
                <td style="color:var(--text4);font-size:11px">{{ (aiPage - 1) * aiPageSize + idx + 1 }}</td>
                <td style="font-size:12px;color:var(--text3);white-space:nowrap">{{ fmtDate(item.created_at) }}</td>
                <td>
                  <span v-if="item.class_id" class="ai-class-chip">Класс #{{ item.class_id }}</span>
                  <span v-else class="ai-class-chip ai-class-general">Общий</span>
                </td>
                <td>
                  <span :class="['ai-type-chip', item.endpoint === 'ai-grade' ? 'ai-type-grade' : 'ai-type-chat']">
                    {{ item.endpoint === 'ai-grade' ? '📋 Проверка' : '💬 Чат' }}
                  </span>
                </td>
                <td style="font-size:12px;color:var(--text3)">{{ getUserEmail(item.user_id) }}</td>
                <td style="text-align:right;font-size:12px;font-variant-numeric:tabular-nums">{{ item.prompt_tokens.toLocaleString() }}</td>
                <td style="text-align:right;font-size:12px;font-variant-numeric:tabular-nums">{{ item.completion_tokens.toLocaleString() }}</td>
                <td style="text-align:right">
                  <span class="token-badge">{{ item.total_tokens.toLocaleString() }}</span>
                </td>
              </tr>
              <tr v-if="!aiLogs.length">
                <td colspan="8" style="text-align:center;padding:32px;color:var(--text4);font-size:13px">
                  <div style="font-size:28px;margin-bottom:8px">🤖</div>
                  Запросов к ИИ пока нет
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pagination -->
        <div v-if="aiTotal > aiPageSize" class="ai-pagination">
          <button class="btn btn-ghost btn-sm" :disabled="aiPage <= 1" @click="loadAiUsage(aiPage - 1)">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M15 18l-6-6 6-6"/></svg>
            Назад
          </button>
          <span style="font-size:13px;color:var(--text3)">
            Стр. {{ aiPage }} / {{ Math.ceil(aiTotal / aiPageSize) }} · {{ aiTotal }} записей
          </span>
          <button class="btn btn-ghost btn-sm" :disabled="aiPage >= Math.ceil(aiTotal / aiPageSize)" @click="loadAiUsage(aiPage + 1)">
            Вперёд
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 18l6-6-6-6"/></svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Create user modal -->
    <div v-if="showCreate" class="overlay" @click.self="showCreate=false">
      <div class="modal anim-scale">
        <div class="modal-head"><span class="modal-title">Создать пользователя</span><button class="btn btn-icon btn-ghost" @click="showCreate=false"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M18 6L6 18M6 6l12 12"/></svg></button></div>
        <div class="frow"><label class="flabel">Email</label><input v-model="nu.e" type="email" class="input" placeholder="user@company.com"/></div>
        <div class="frow"><label class="flabel">Пароль</label><input v-model="nu.p" type="password" class="input" placeholder="Минимум 6 символов"/></div>
        <div class="frow"><label class="flabel">Роль</label><select v-model="nu.r" class="input"><option value="student">Студент</option><option value="teacher">Учитель</option><option value="admin">Администратор</option></select></div>
        <div class="modal-foot">
          <button class="btn btn-white" @click="showCreate=false">Отмена</button>
          <button class="btn btn-blue" :disabled="!nu.e||!nu.p||crU" @click="createU">
            <div v-if="crU" class="spinner" style="width:13px;height:13px;border-width:2px;border-color:rgba(255,255,255,.3);border-top-color:#fff"></div><span v-else>Создать</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useAuthStore } from '~/stores/auth.store'
import { useToast } from '~/composables/useToast/useToast'
import { useAdminSvc } from '~/services/admin'
import { useChatsSvc } from '~/services/chats'
import { usePostsSvc } from '~/services/posts'
import { useI18n } from '~/composables/usel18n/useI18n'
definePageMeta({ layout: 'default' })
const auth = useAuthStore(); const toast = useToast(); const adminSvc = useAdminSvc()
const { t, lang } = useI18n()
const chatsSvc = useChatsSvc(); const postsSvc = usePostsSvc()
const tab = ref('users'); const users = ref<any[]>([]); const loadingU = ref(false); const sq = ref(''); const showCreate = ref(false); const crU = ref(false)
const togglingAi = ref<Record<number, boolean>>({})
const nu = ref({ e: '', p: '', r: 'student' }); const chatsCount = ref(0); const classesCount = ref(0)
const today = new Date().toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' }).replace(/\//g, '.')
const avColors = ['bg-b0', 'bg-b1', 'bg-b2', 'bg-b3', 'bg-b4', 'bg-b5']
const colorFor = (id: number) => avColors[id % avColors.length]
const fUsers = computed(() => users.value.filter(u => {
  const q = sq.value.toLowerCase()
  return u.email.toLowerCase().includes(q) || (u.full_name || '').toLowerCase().includes(q)
}))

// ── AI Usage ──────────────────────────────────────────────────────────────────
const aiLogs = ref<any[]>([]); const aiSummary = ref<any[]>([]); const aiTotal = ref(0)
const aiPage = ref(1); const aiPageSize = 50; const aiLoading = ref(false)
const aiFilterClass = ref<number | null>(null)
const aiTotalTokens = computed(() => aiSummary.value.filter((s: any) => aiFilterClass.value === null || s.class_id === aiFilterClass.value).reduce((sum: number, s: any) => sum + (s.total_tokens || 0), 0))
const userMap = computed(() => { const m: Record<number, string> = {}; for (const u of users.value) m[u.id] = u.full_name || u.email; return m })
const getUserEmail = (uid: number | null) => uid ? (userMap.value[uid] || '#' + uid) : '—'
const fmtDate = (iso: string) => { if (!iso) return '—'; try { const d = new Date(iso); return d.toLocaleDateString('ru-RU', { day: '2-digit', month: '2-digit', year: '2-digit' }) + ' ' + d.toLocaleTimeString('ru-RU', { hour: '2-digit', minute: '2-digit' }) } catch { return iso } }
const loadAiUsage = async (page = 1) => { aiLoading.value = true; aiPage.value = page; try { const params: any = { page, page_size: aiPageSize }; if (aiFilterClass.value !== null) params.class_id = aiFilterClass.value; const res = await adminSvc.aiUsage(params); aiLogs.value = res.items; aiTotal.value = res.total } catch { toast.err('Ошибка загрузки данных ИИ') } finally { aiLoading.value = false } }
const loadAiSummary = async () => { try { aiSummary.value = await adminSvc.aiUsageSummary() } catch {} }
const setClassFilter = (classId: number | null) => { aiFilterClass.value = aiFilterClass.value === classId ? null : classId; loadAiUsage(1) }
const switchToAiUsage = () => { tab.value = 'ai-usage'; if (!aiLogs.value.length && !aiLoading.value) loadAiUsage(1) }

// ── User actions ──────────────────────────────────────────────────────────────
const toggleAiUnlimited = async (u: any) => {
  togglingAi.value = { ...togglingAi.value, [u.id]: true }
  const newVal = !u.ai_unlimited
  try {
    await adminSvc.setAiUnlimited(u.id, newVal)
    u.ai_unlimited = newVal
    toast.ok(newVal ? 'ИИ доступ открыт' : 'ИИ доступ ограничен')
  } catch { toast.err('Ошибка обновления ИИ доступа') }
  finally { togglingAi.value = { ...togglingAi.value, [u.id]: false } }
}

const setRole = async (id: number, r: string) => { try { await adminSvc.role(id, r); const u = users.value.find(u => u.id === id); if (u) u.role = r; toast.ok('Роль обновлена') } catch { toast.err('Ошибка') } }
const doBlock = async (id: number) => { try { await adminSvc.block(id); const u = users.value.find(u => u.id === id); if (u) u.is_active = false; toast.ok('Заблокирован') } catch { toast.err('Ошибка') } }
const doUnblock = async (id: number) => { try { await adminSvc.unblock(id); const u = users.value.find(u => u.id === id); if (u) u.is_active = true; toast.ok('Разблокирован') } catch { toast.err('Ошибка') } }
const doDel = async (id: number) => { try { await adminSvc.del(id); users.value = users.value.filter(u => u.id !== id); toast.ok('Удалён') } catch { toast.err('Ошибка') } }
const createU = async () => { crU.value = true; try { const u = await adminSvc.create({ email: nu.value.e, password: nu.value.p, role: nu.value.r }); users.value.unshift(u); showCreate.value = false; nu.value = { e: '', p: '', r: 'student' }; toast.ok('Создан') } catch (e: any) { toast.err(e?.response?.data?.detail || 'Ошибка') } finally { crU.value = false } }
onMounted(async () => {
  if (!auth.isAdmin) return
  loadingU.value = true
  try { users.value = await adminSvc.users() } catch {} finally { loadingU.value = false }
  try { const c = await chatsSvc.list(); chatsCount.value = c.length } catch {}
  try { const p = await postsSvc.list(); classesCount.value = p.filter((x: any) => { try { return JSON.parse(x.body).type === 'class' } catch { return false } }).length } catch {}
  loadAiSummary()
})
</script>
<style scoped src="~/assets/pages/admin.css"></style>