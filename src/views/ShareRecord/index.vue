<script lang="ts" setup>
defineOptions({
  // 命名当前组件
  name: 'ShareRecord',
})

import { reactive, ref, watch, onBeforeMount } from 'vue'

import { useI18n } from 'vue-i18n'

const { t } = useI18n()

import { type GetTableData } from '@/api/table/types/table'
import { type FormInstance, type FormRules, ElMessage, ElMessageBox } from 'element-plus'
import { usePagination } from '@/hooks/usePagination'
import api from '@/utils/api'
import { useUserStore } from '@/store/modules/user'

import { toLocalTime } from '@/utils/index'

const loading = ref<boolean>(false)
const { paginationData, handleCurrentChange, handleSizeChange } = usePagination()

const tableData = ref<GetTableData[]>([])
const searchFormRef = ref<FormInstance | null>(null)

const userStore = useUserStore()

const drawer = ref(false)

onBeforeMount(async () => {
  localStorage.setItem('inviteWalletAddress', userStore.walletAddress)
})

const searchData = reactive({
  pAccount: useUserStore().walletAddress,
  column: 'createTime',
  order: 'desc',
})
const getTableData = async () => {
  loading.value = true

  searchData.pageNo = paginationData.currentPage
  searchData.pageSize = paginationData.pageSize

  const { result } = await api.get('/mgn/inviteRecord/childList', searchData)
  tableData.value = result.records
  paginationData.total = result.total
  loading.value = false
}
const handleSearch = () => {
  paginationData.currentPage === 1 ? getTableData() : (paginationData.currentPage = 1)
}
const resetSearch = () => {
  searchFormRef.value?.resetFields()
  handleSearch()
}

async function loadNode(node, treeNode, resolve) {
  const level = await getlevel(node.id)

  if (level > 2) {
    node.loading = false
    return resolve([])
  } else {
    const { result } = await api.get('/mgn/inviteRecord/childList', {
      pageNo: node.pageNo,
      pageSize: -1,
      pid: node.id,
    })

    node.total = result.total
    node.pages = result.pages

    return resolve(result.records)
  }
}

async function getlevel(recordId) {
  const { result } = await api.get('/mgn/inviteRecord/getlevel', {
    pAccount: useUserStore().walletAddress,
    recordId: recordId,
  })

  return result
}

function showDrawer() {
  drawer.value = true
}

defineExpose({
  showDrawer,
})

//#endregion

/** 监听分页参数的变化 */
watch([() => paginationData.currentPage, () => paginationData.pageSize], getTableData, { immediate: true })
</script>


<template>
  <el-drawer v-model="drawer" size="70%" title="My Invitation Records">
    <template #title>
      <h4>My Invitation Records</h4>
    </template>

    <div class="app-container">
      <!-- 移除滚动容器的样式 -->
      <div class="table-container">
        <el-card v-loading="loading" shadow="never" class="search-wrapper">
          <el-form ref="searchFormRef" :inline="true" :model="searchData"> </el-form>
        </el-card>

        <el-card v-loading="loading" shadow="never">
          <div class="toolbar-wrapper">
            <div>
              <el-tooltip content="下载">
                <!-- <el-button type="primary" :icon="Download" circle /> -->
              </el-tooltip>
              <el-tooltip content="刷新当前页">
                <!-- <el-button type="primary" :icon="RefreshRight" circle @click="getTableData" /> -->
              </el-tooltip>
            </div>
          </div>
          <div class="table-wrapper">
            <el-table :data="tableData" lazy row-key="id" :load="loadNode" :tree-props="{ children: 'children', hasChildren: 'hasChild' }">
              <el-table-column prop="walletAddress" :label="t('shareRecord.walletAddress')" align="center"> </el-table-column>

              <el-table-column prop="createTime" :label="t('shareRecord.createTime')" align="center">
                <template #default="{ row }">{{ toLocalTime(row.createTime) }}</template>
              </el-table-column>

              <!-- your columns here -->
            </el-table>
          </div>
          <div class="pager-wrapper">
            <el-pagination background :layout="paginationData.layout" :page-sizes="paginationData.pageSizes" :total="paginationData.total" :page-size="paginationData.pageSize" :currentPage="paginationData.currentPage" @size-change="handleSizeChange" @current-change="handleCurrentChange" />
          </div>
        </el-card>
      </div>
    </div>
  </el-drawer>
</template>

<style lang="scss" scoped>
.search-wrapper {
  margin-bottom: 20px;
  :deep(.el-card__body) {
    padding-bottom: 2px;
  }
}

.toolbar-wrapper {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.table-wrapper {
  margin-bottom: 20px;
}

.pager-wrapper {
  display: flex;
  justify-content: flex-end;
}
</style>


