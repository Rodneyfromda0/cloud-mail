<template>
  <div class="inbox-workspace">
    <emailScroll ref="scroll"
               :cancel-success="cancelStar"
               :star-success="addStar"
               :getEmailList="getEmailList"
               :emailDelete="deleteMessages"
               :star-add="starAdd"
               :star-cancel="starCancel"
               :email-read="markRead"
               :mailbox-actions="mailboxActions"
               :mailbox-action="applyMailboxAction"
               :selected-email-id="emailStore.contentData.email?.emailId || 0"
               :show-unread="true"
               actionLeft="4px"
               @jump="jumpContent"
    />
    <section class="reading-pane" aria-label="Email reader">
      <ContentView
        v-if="emailStore.contentData.email"
        embedded
        @close="closeReader"
      />
      <el-empty v-else description="Select an email to read" />
    </section>
  </div>
</template>

<script setup>
import {useAccountStore} from "@/store/account.js";
import {useEmailStore} from "@/store/email.js";
import {useSettingStore} from "@/store/setting.js";
import emailScroll from "@/components/email-scroll/index.vue"
import {emailList, emailDelete, emailLatest, emailRead} from "@/request/email.js";
import {starAdd, starCancel} from "@/request/star.js";
import {mailboxFolderList, mailboxMove} from "@/request/mailbox.js";
import {computed, defineOptions, onMounted, onUnmounted, ref, watch} from "vue";
import {sleep} from "@/utils/time-utils.js";
import router from "@/router/index.js";
import { useRoute } from 'vue-router'
import ContentView from '@/views/content/index.vue'

defineOptions({
  name: 'email'
})

const route = useRoute();
const emailStore = useEmailStore();
const accountStore = useAccountStore();
const settingStore = useSettingStore();
const scroll = ref({})
const isDesktop = ref(window.innerWidth >= 1024)
const mailboxActions = ref([
  {label: 'Archive', value: 'archive'},
  {label: 'Move to Spam', value: 'spam'},
  {label: 'Move to Trash', value: 'trash'},
  {label: 'Snooze for 1 hour', value: 'snooze-hour'},
  {label: 'Snooze until tomorrow', value: 'snooze-tomorrow'},
  {label: 'Mark as Newsletter', value: 'category:newsletter'},
  {label: 'Mark as Notification', value: 'category:notification'},
  {label: 'Mark as Primary', value: 'category:primary'},
])
onMounted(() => {
  emailStore.emailScroll = scroll;
  window.addEventListener('resize', updateDesktopLayout)
  latest()
  mailboxFolderList().then(folders => {
    mailboxActions.value.push(...folders.map(folder => ({
      label: `Move to ${folder.name}`,
      value: `custom:${folder.folderId}`,
    })))
  }).catch(error => console.error(error))
})

onUnmounted(() => {
  window.removeEventListener('resize', updateDesktopLayout)
})

function updateDesktopLayout() {
  isDesktop.value = window.innerWidth >= 1024
}


watch(() => accountStore.currentAccountId, () => {
  scroll.value.refreshList();
})

watch(
  () => emailStore.searchParams,
  () => {
    scroll.value.refreshList()
  },
  {deep: true}
)

function jumpContent(email) {
  emailStore.contentData.email = emailStore.toContentEmail(email)
  emailStore.contentData.delType = 'logic'
  emailStore.contentData.showUnread = true
  emailStore.contentData.showStar = true
  emailStore.contentData.showReply = true
  if (!isDesktop.value) router.push('/messages')
}

function closeReader() {
  emailStore.contentData.email = null
  emailStore.contentData.showUnread = false
}

const existIds = new Set();

async function latest() {
  while (true) {

    let autoRefresh = settingStore.settings.autoRefresh;
    await sleep(autoRefresh > 1 ? autoRefresh * 1000 : 3000);

    if (route.name !== 'email') {
      continue;
    }

    const searchParams = emailStore.searchParams
    if (searchParams.text || searchParams.startDate || searchParams.endDate) {
      continue
    }

    const latestId = scroll.value.latestEmail?.emailId

    if (!scroll.value.firstLoad && autoRefresh > 1) {
      try {
        const accountId = accountStore.currentAccountId
        const allReceive = scroll.value.latestEmail?.allReceive
        let list = []

        //确保发起请求时最后一个邮件是当前账号的,或者
        if (accountId === scroll.value.latestEmail?.reqAccountId) {
          list = await emailLatest(latestId, accountId, allReceive);
        }

        //确保请求回来后，账号没有切换，时间排序没有改变，全部邮件类型没变
        if (accountId === accountStore.currentAccountId && allReceive === accountStore.currentAccount.allReceive) {
          if (list.length > 0) {
            emailStore.applyFullList(list)

            for (let email of list) {

              email.reqAccountId = accountId;
              email.allReceive = allReceive;

              if (!existIds.has(email.emailId)) {

                existIds.add(email.emailId)
                scroll.value.addItem(email)

                await sleep(50)
              }

            }
            window.dispatchEvent(new Event('mailbox:updated'))

          }

        }
      } catch (e) {
        if (e.code === 401 || e.code === 403) {
          settingStore.settings.autoRefresh = 0;
        }
        console.error(e)
      }
    }
  }
}

function addStar(email) {
  emailStore.starScroll?.addItem(email)
}

function cancelStar(email) {
  emailStore.starScroll?.deleteEmail([email.emailId])
}

function getEmailList(emailId, size) {
  const accountId =  accountStore.currentAccountId;
  const allReceive = accountStore.currentAccount.allReceive;
  return emailStore.fetchList(full =>
    emailList(accountId, allReceive, emailId, 0, size, 0, full, emailStore.searchParams)
  ).then(data => {
    data.latestEmail.reqAccountId = accountId;
    data.latestEmail.allReceive = allReceive;
    return data;
  })
}

function markRead(ids) {
  return emailRead(ids).then(result => {
    window.dispatchEvent(new Event('mailbox:updated'))
    return result
  })
}

function deleteMessages(ids) {
  return emailDelete(ids).then(result => {
    window.dispatchEvent(new Event('mailbox:updated'))
    return result
  })
}

async function applyMailboxAction(ids, action) {
  if (action.startsWith('category:')) {
    await mailboxMove(ids, {category: action.slice('category:'.length)})
  } else if (action.startsWith('custom:')) {
    await mailboxMove(ids, {customFolderId: Number(action.slice('custom:'.length))})
  } else if (action === 'snooze-hour' || action === 'snooze-tomorrow') {
    const delay = action === 'snooze-hour' ? 60 * 60 * 1000 : 24 * 60 * 60 * 1000
    await mailboxMove(ids, 'snoozed', new Date(Date.now() + delay).toISOString())
  } else {
    await mailboxMove(ids, action)
  }
  window.dispatchEvent(new Event('mailbox:updated'))
}

</script>
<style>
.icon {
  cursor: pointer;
}
</style>
<style scoped>
.inbox-workspace {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  height: 100%;
  min-height: 0;
  overflow: hidden;
}

.reading-pane {
  min-width: 0;
  min-height: 0;
  overflow: hidden;
  border-left: 1px solid var(--el-border-color);
}

.reading-pane :deep(.el-empty) {
  height: 100%;
}

@media (max-width: 1023px) {
  .inbox-workspace {
    display: block;
  }

  .reading-pane {
    display: none;
  }
}
</style>
