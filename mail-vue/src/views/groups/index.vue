<template>
  <div class="groups-page">
    <div class="page-header">
      <div>
        <h2>Contact groups</h2>
        <p>Reusable recipient lists are available from the composer.</p>
      </div>
      <el-button type="primary" @click="startNew">Create a group</el-button>
    </div>
    <el-table :data="groups" v-loading="loading">
      <el-table-column prop="name" label="Group" />
      <el-table-column label="Recipients" width="140">
        <template #default="{row}">{{ row.emails.length }}</template>
      </el-table-column>
      <el-table-column label="Actions" width="180">
        <template #default="{row}">
          <el-button text @click="edit(row)">Edit</el-button>
          <el-button type="danger" text @click="remove(row)">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog v-model="dialogVisible" :title="form.groupId ? 'Edit group' : 'Create a group'" width="min(640px, 95vw)">
      <el-form label-position="top">
        <el-form-item label="Group name">
          <el-input v-model="form.name" maxlength="60" />
        </el-form-item>
        <el-form-item label="Email addresses (comma or line separated)">
          <el-input v-model="form.addresses" type="textarea" :rows="6" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" :loading="saving" @click="save">Save group</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {onMounted, reactive, ref} from 'vue'
import {ElMessage, ElMessageBox} from 'element-plus'
import {
  mailboxGroupCreate,
  mailboxGroupDelete,
  mailboxGroupUpdate,
  mailboxGroups,
} from '@/request/mailbox.js'

const groups = ref([])
const loading = ref(false)
const saving = ref(false)
const dialogVisible = ref(false)
const form = reactive({groupId: 0, name: '', addresses: ''})

async function loadGroups() {
  loading.value = true
  try {
    groups.value = await mailboxGroups()
  } finally {
    loading.value = false
  }
}

function startNew() {
  Object.assign(form, {groupId: 0, name: '', addresses: ''})
  dialogVisible.value = true
}

function edit(group) {
  Object.assign(form, {
    groupId: group.groupId,
    name: group.name,
    addresses: group.emails.join('\n'),
  })
  dialogVisible.value = true
}

async function save() {
  const emails = form.addresses.split(/[,\n;]/).map(email => email.trim()).filter(Boolean)
  saving.value = true
  try {
    if (form.groupId) {
      await mailboxGroupUpdate(form.groupId, {name: form.name, emails})
    } else {
      await mailboxGroupCreate({name: form.name, emails})
    }
    dialogVisible.value = false
    await loadGroups()
  } finally {
    saving.value = false
  }
}

async function remove(group) {
  try {
    await ElMessageBox.confirm(
      `Delete the "${group.name}" contact group?`,
      'Delete group',
      {type: 'warning', confirmButtonText: 'Delete', cancelButtonText: 'Cancel'}
    )
  } catch (action) {
    if (action === 'cancel' || action === 'close') return
    throw action
  }
  await mailboxGroupDelete(group.groupId)
  await loadGroups()
  ElMessage({message: 'Group deleted.', type: 'success'})
}

onMounted(() => loadGroups())
</script>

<style scoped>
.groups-page {
  height: 100%;
  overflow: auto;
  padding: 24px;
}

.page-header {
  display: flex;
  align-items: start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 18px;
}

.page-header h2 {
  margin: 0 0 8px;
}

.page-header p {
  margin: 0;
  color: var(--el-text-color-secondary);
}
</style>
