<template>
  <el-scrollbar class="scroll" role="navigation" aria-label="Navigation sidebar">
    <div>
      <div class="title">
        <img class="brand-logo" src="/xpass-logo.png" alt="XPass" />
      </div>
      <el-button
        v-perm="'email:send'"
        class="compose-cta"
        type="primary"
        aria-label="Send mail (C)"
        @click="openCompose"
      >
        <Icon icon="cil:send" width="20" height="20" />
        <span>SEND MAIL</span>
        <kbd aria-hidden="true">C</kbd>
      </el-button>
      <el-menu :collapse="false" text-color="#fff" active-text-color="#fff">
        <div class="section-title">QUICK FILTERS</div>
        <el-menu-item @click="router.push({name: 'email'})" index="home">
          <Icon icon="solar:home-2-line-duotone" width="20" height="20" />
          <span class="menu-name">Home</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'mail-folder', params: {folderKey: 'all-unread'}})" index="all-unread"
                      :class="route.params.folderKey === 'all-unread' ? 'choose-item' : ''">
          <Icon icon="fluent:mail-unread-20-regular" width="20" height="20" />
          <span class="menu-name">All unread</span>
          <span v-if="counts.unread" class="count-badge">{{ badge(counts.unread) }}</span>
        </el-menu-item>
        <el-menu-item @click="openGroupDialog" index="create-group">
          <Icon icon="fluent:people-team-add-20-regular" width="20" height="20" />
          <span class="menu-name">Create a group</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'groups'})" index="groups"
                      :class="route.name === 'groups' ? 'choose-item' : ''">
          <Icon icon="fluent:people-team-20-regular" width="20" height="20" />
          <span class="menu-name">Manage groups</span>
        </el-menu-item>
        <div class="folder-tools">
          <span>FOLDERS</span>
          <div>
            <el-button text aria-label="Create new folder" @click="showFolderDialog = true">
              <Icon icon="fluent:add-20-regular" width="18" height="18" />
            </el-button>
            <el-button text aria-label="Manage folders" @click="router.push({name: 'folder-settings'})">
              <Icon icon="fluent:settings-48-regular" width="18" height="18" />
            </el-button>
          </div>
        </div>
        <el-menu-item @click="router.push({name: 'email'})" index="email"
                      :class="route.meta.name === 'email' ? 'choose-item' : ''">
          <Icon icon="hugeicons:mailbox-01" width="20" height="20" />
          <span class="menu-name">{{$t('inbox')}}</span>
          <span v-if="counts.inbox" class="count-badge">{{ badge(counts.inbox) }}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'send'})" index="send" v-perm="'email:send'"
                      :class="route.meta.name === 'send' ? 'choose-item' : ''">
          <Icon icon="cil:send" width="20" height="20" />
          <span class="menu-name">{{$t('sent')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'draft'})" index="draft" v-perm="'email:send'"
                      :class="route.meta.name === 'draft' ? 'choose-item' : ''">
          <Icon icon="ep:document" width="19" height="19" />
          <span class="menu-name">{{$t('drafts')}}</span>
          <span v-if="draftCount" class="count-badge">{{ badge(draftCount) }}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'templates'})" index="templates"
                      :class="route.name === 'templates' ? 'choose-item' : ''">
          <Icon icon="fluent:document-bullet-list-20-regular" width="20" height="20" />
          <span class="menu-name">Templates</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('snoozed')" index="snoozed"
                      :class="route.params.folderKey === 'snoozed' ? 'choose-item' : ''">
          <Icon icon="fluent:clock-20-regular" width="20" height="20" />
          <span class="menu-name">Snoozed</span>
          <span v-if="counts.snoozed" class="count-badge">{{ badge(counts.snoozed) }}</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('spam')" index="spam"
                      :class="route.params.folderKey === 'spam' ? 'choose-item' : ''">
          <Icon icon="fluent:shield-error-20-regular" width="20" height="20" />
          <span class="menu-name">Spam</span>
          <span v-if="counts.spam" class="count-badge">{{ badge(counts.spam) }}</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('trash')" index="trash"
                      :class="route.params.folderKey === 'trash' ? 'choose-item' : ''">
          <Icon icon="fluent:delete-20-regular" width="20" height="20" />
          <span class="menu-name">Trash</span>
          <span v-if="counts.trash" class="count-badge">{{ badge(counts.trash) }}</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('archive')" index="archive"
                      :class="route.params.folderKey === 'archive' ? 'choose-item' : ''">
          <Icon icon="fluent:archive-20-regular" width="20" height="20" />
          <span class="menu-name">Archive</span>
          <span v-if="counts.archive" class="count-badge">{{ badge(counts.archive) }}</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('outbox')" index="outbox"
                      :class="route.params.folderKey === 'outbox' ? 'choose-item' : ''">
          <Icon icon="fluent:send-clock-20-regular" width="20" height="20" />
          <span class="menu-name">Outbox</span>
          <span v-if="counts.outbox" class="count-badge">{{ badge(counts.outbox) }}</span>
        </el-menu-item>
        <el-menu-item v-for="folder in folders" :key="folder.folderId"
                      @click="openFolder('custom-' + folder.folderId)" :index="'custom-' + folder.folderId"
                      :class="route.params.folderKey === 'custom-' + folder.folderId ? 'choose-item' : ''">
          <Icon icon="fluent:folder-20-regular" width="20" height="20" />
          <span class="menu-name">{{ folder.name }}</span>
          <span v-if="folder.unreadCount" class="count-badge">{{ badge(folder.unreadCount) }}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'star'})" index="star"
                      :class="route.meta.name === 'star' ? 'choose-item' : ''">
          <Icon icon="solar:star-line-duotone" width="20" height="20" />
          <span class="menu-name" style="margin-left: 16px">{{$t('starred')}}</span>
        </el-menu-item>
        <div class="section-title">SMART CATEGORIES</div>
        <el-menu-item @click="openFolder('newsletter')" index="newsletter"
                      :class="route.params.folderKey === 'newsletter' ? 'choose-item' : ''">
          <Icon icon="fluent:mail-list-20-regular" width="20" height="20" />
          <span class="menu-name">Newsletter</span>
          <span v-if="counts.newsletter" class="count-badge">{{ badge(counts.newsletter) }}</span>
        </el-menu-item>
        <el-menu-item @click="openFolder('notification')" index="notification"
                      :class="route.params.folderKey === 'notification' ? 'choose-item' : ''">
          <Icon icon="fluent:alert-20-regular" width="20" height="20" />
          <span class="menu-name">Notification</span>
          <span v-if="counts.notification" class="count-badge">{{ badge(counts.notification) }}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'setting'})" index="setting"
                      :class="route.meta.name === 'setting' ? 'choose-item' : ''">
          <Icon icon="fluent:settings-48-regular" width="20" height="20" />
          <span class="menu-name" style="margin-left: 16px">{{$t('settings')}}</span>
        </el-menu-item>
        <div class="manage-title" v-perm="['all-email:query','user:query','role:query','setting:query','analysis:query','reg-key:query']">
          <div>{{$t('manage')}}</div>
        </div>
        <el-menu-item @click="router.push({name: 'analysis'})" index="analysis" v-perm="'analysis:query'"
                      :class="route.meta.name === 'analysis' ? 'choose-item' : ''">
          <Icon icon="fluent:data-pie-20-regular" width="24" height="24" />
          <span class="menu-name" style="margin-left: 13px">{{$t('analytics')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'user'})" index="user" v-perm="'user:query'"
                      :class="route.meta.name === 'user' ? 'choose-item' : ''">
          <Icon icon="si:user-alt-2-line" width="20" height="20" />
          <span class="menu-name" style="margin-left: 16px">{{$t('allUsers')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'all-email'})" index="all-email" v-perm="'all-email:query'"
                      :class="route.meta.name === 'all-email' ? 'choose-item' : ''">
          <Icon icon="fluent:mail-list-28-regular" width="22" height="22" />
          <span class="menu-name" style="margin-left: 15px">{{$t('allMail')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'role'})" index="role" v-perm="'role:query'"
                      :class="route.meta.name === 'role' ? 'choose-item' : ''">
          <Icon icon="fluent:lock-closed-16-regular" width="22" height="22" />
          <span class="menu-name" style="margin-left: 15px">{{$t('permissions')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'reg-key'})" index="reg-key" v-perm="'reg-key:query'"
                      :class="route.meta.name === 'reg-key' ? 'choose-item' : ''">
          <Icon icon="fluent:fingerprint-20-filled" width="22" height="22" />
          <span class="menu-name" style="margin-left: 15px">{{$t('inviteCode')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'sys-setting'})" index="sys-setting" v-perm="'setting:query'"
                      :class="route.meta.name === 'sys-setting' ? 'choose-item' : ''">
          <Icon icon="eos-icons:system-ok-outlined" width="18" height="18" style="margin-left: 2px" />
          <span class="menu-name" style="margin-left: 17px">{{$t('SystemSettings')}}</span>
        </el-menu-item>
      </el-menu>
    </div>
    <el-dialog v-model="showFolderDialog" title="Create new folder" width="420px">
      <el-input v-model="folderName" maxlength="60" show-word-limit placeholder="Folder name" @keyup.enter="createFolder" />
      <template #footer>
        <el-button @click="showFolderDialog = false">{{ $t('cancel') }}</el-button>
        <el-button type="primary" :loading="saving" @click="createFolder">Create</el-button>
      </template>
    </el-dialog>
    <el-dialog v-model="showGroupDialog" title="Create contact group" width="520px">
      <el-form label-position="top">
        <el-form-item label="Group name">
          <el-input v-model="groupForm.name" maxlength="60" />
        </el-form-item>
        <el-form-item label="Email addresses (comma or line separated)">
          <el-input v-model="groupForm.emails" type="textarea" :rows="4" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showGroupDialog = false">{{ $t('cancel') }}</el-button>
        <el-button type="primary" :loading="saving" @click="createGroup">Create</el-button>
      </template>
    </el-dialog>
  </el-scrollbar>
</template>

<script setup>
import router from "@/router/index.js";
import { useRoute } from "vue-router";
import {Icon} from "@iconify/vue";
import {useUiStore} from "@/store/ui.js";
import {onMounted, onUnmounted, ref, watch} from "vue";
import {mailboxCounts, mailboxFolderCreate, mailboxFolderList, mailboxGroupCreate} from "@/request/mailbox.js";
import {userDraftStore} from "@/store/draft.js";
import db from "@/db/db.js";

const uiStore = useUiStore();
const route = useRoute();
const draftStore = userDraftStore();
const folders = ref([]);
const counts = ref({});
const showFolderDialog = ref(false);
const showGroupDialog = ref(false);
const folderName = ref('');
const groupForm = ref({name: '', emails: ''});
const saving = ref(false);
const draftCount = ref(0);
let countRefreshTimer;

async function refreshNavigation() {
  const [folderRows, folderCounts] = await Promise.all([mailboxFolderList(), mailboxCounts()]);
  folders.value = folderRows;
  counts.value = folderCounts;
}

function refreshAfterMailboxUpdate() {
  refreshNavigation().catch(error => console.error(error));
}

function openFolder(folderKey) {
  router.push({name: 'mail-folder', params: {folderKey}});
}

function badge(value) {
  const count = Number(value) || 0;
  return count > 99 ? '99+' : count;
}

function openGroupDialog() {
  groupForm.value = {name: '', emails: ''};
  showGroupDialog.value = true;
}

async function createFolder() {
  if (!folderName.value.trim()) return;
  saving.value = true;
  try {
    await mailboxFolderCreate(folderName.value);
    folderName.value = '';
    showFolderDialog.value = false;
    await refreshNavigation();
  } finally {
    saving.value = false;
  }
}

async function createGroup() {
  const emails = groupForm.value.emails.split(/[,\n;]/).map(email => email.trim()).filter(Boolean);
  saving.value = true;
  try {
    await mailboxGroupCreate({name: groupForm.value.name, emails});
    showGroupDialog.value = false;
  } finally {
    saving.value = false;
  }
}

async function updateDraftCount() {
  if (db.value?.draft) {
    draftCount.value = await db.value.draft.count();
  }
}

onMounted(() => {
  refreshNavigation().catch(error => console.error(error));
  updateDraftCount().catch(error => console.error(error));
  window.addEventListener('mailbox:updated', refreshAfterMailboxUpdate);
  countRefreshTimer = window.setInterval(refreshAfterMailboxUpdate, 30000);
});

onUnmounted(() => {
  window.removeEventListener('mailbox:updated', refreshAfterMailboxUpdate);
  window.clearInterval(countRefreshTimer);
});
watch(() => draftStore.refreshList, () => updateDraftCount().catch(error => console.error(error)));
watch(() => db.value, () => updateDraftCount().catch(error => console.error(error)));
watch(() => route.path, () => refreshNavigation().catch(error => console.error(error)));

function openCompose() {
  uiStore.writerRef?.open()
}

</script>

<style lang="scss" scoped>

.title {
  margin: 14px 18px 18px;
  height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.brand-logo {
  width: 154px;
  height: 52px;
  object-fit: contain;
}

.compose-cta {
  display: flex;
  width: calc(100% - 36px);
  height: 46px;
  margin: 0 18px 20px;
  justify-content: flex-start;
  gap: 12px;
  padding: 0 14px;
  border: 0;
  border-radius: 11px;
  background: #ff5500;
  color: #fff;
  box-shadow: 0 5px 16px rgba(255, 85, 0, .2);
  font-size: 13px;
  font-weight: 800;
  letter-spacing: .08em;
  transition: filter .15s ease, transform .15s ease;
}

.compose-cta:hover {
  background: #e94c00;
  filter: brightness(1.08);
  transform: translateY(-1px);
}

.compose-cta:active {
  transform: scale(.98);
}

.compose-cta:focus-visible {
  outline: 2px solid #fff;
  outline-offset: 3px;
}

.compose-cta kbd {
  margin-left: auto;
  padding: 2px 7px;
  border: 1px solid rgba(255, 255, 255, .35);
  border-radius: 5px;
  color: rgba(255, 255, 255, .8);
  font-family: inherit;
  font-size: 11px;
  font-weight: 600;
  line-height: 1.4;
  letter-spacing: 0;
}

.section-title {
  margin: 0 20px 8px;
  padding-top: 10px;
  border-top: 1px solid #302a26;
  color: #a99b91;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: .14em;
}

.folder-tools {
  margin: 0 14px 6px 20px;
  padding-top: 10px;
  border-top: 1px solid #302a26;
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #a99b91;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: .14em;
}

.folder-tools :deep(.el-button) {
  color: #a99b91;
  padding: 4px;
}

.count-badge {
  margin-left: auto;
  padding: 2px 7px;
  border-radius: 999px;
  background: rgba(255, 85, 0, .16);
  color: #ff8751;
  font-size: 11px;
  font-weight: 700;
  line-height: 1.4;
}

.manage-title {
  margin-top: 10px;
  padding-left: 20px;
  color: #a99b91;
}

.el-menu-item {
  margin: 3px 10px !important;
  border-radius: 8px;
  height: 40px;
  padding: 10px !important;
  color: #c1b4aa;
}

.choose-item {
  position: relative;
  font-weight: 400;
  color: #fff0e7 !important;
  background: rgba(255, 85, 0, 0.15) !important;
  backdrop-filter: blur(4px);
}

.choose-item::before {
  position: absolute;
  top: 8px;
  bottom: 8px;
  left: 0;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: #ff5500;
  content: "";
}

@media (hover: hover) {
  .el-menu-item:hover {
    background: rgba(255, 255, 255, 0.07) !important;
    color: #fff !important;
  }
}

.menu-name {
  margin-left: 16px;
  user-select: none;
}


:deep(.el-scrollbar__wrap--hidden-default ) {
  background: #1d1a18 !important;
}

:deep(.el-menu-item) {
  background: #1d1a18;
}

:deep(.el-menu) {
  background: #1d1a18;
}

.el-menu {
  border-right: 0;
  width: 100%;
}

.scroll,
:deep(.el-scrollbar__view) {
  background: #1d1a18;
}

:deep(.el-divider__text) {
  background: var(--aside-backgound);
  color: #FFFFFF;
}

.scroll {
  height: 100%;
}

.aside {
  width: 260px !important;
  height: 100%;
  flex: 0 0 260px;
  background: #1d1a18;
}
</style>
