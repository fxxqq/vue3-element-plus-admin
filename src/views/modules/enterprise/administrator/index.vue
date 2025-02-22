<template>
  <ContainerSidebar ref="refContainerSidebar" :scroll="false">
    <template #sidebar>
      <EnterpriseSidebar v-model="active" @change="changeHandle" />
    </template>
    <template #header>
      <el-form ref="refForm" :inline="true" @keyup.enter="reacquireHandle()">
        <el-form-item>
          <el-input v-model="form.name" placeholder="用户名/昵称" clearable />
        </el-form-item>
        <el-form-item>
          <el-input v-model="form.department" placeholder="部门" clearable />
        </el-form-item>
        <el-form-item>
          <el-date-picker
            v-model="form.date"
            type="daterange"
            range-separator="-"
            start-placeholder="创建开始日期"
            end-placeholder="创建结束日期"
            clearable />
        </el-form-item>
        <el-form-item>
          <el-button v-repeat @click="reacquireHandle()" :disabled="!active">搜索</el-button>
          <el-button v-repeat @click="clearJson(form), reacquireHandle()" :disabled="!active">重置</el-button>
          <el-button
            v-permission="'global:administrator:create'"
            type="primary"
            @click="addEditHandle()"
            :disabled="!active">新增</el-button>
          <el-button
            v-permission="'global:administrator:delete'"
            type="danger"
            :disabled="selection.length <= 0 || !active"
            @click="deleteHandle()">批量删除</el-button>
        </el-form-item>
      </el-form>
    </template>
    <template #default>
      <el-table
        ref="refTable"
        v-loading="loading"
        :data="list"
        @selection-change="selectionHandle"
        border>
        <el-table-column align="center" type="selection" width="50" />
        <el-table-column
          align="center"
          label="ID"
          prop="id"
          width="80" />
        <el-table-column
          align="center"
          label="头像"
          prop="username"
          width="80">
          <template v-slot="{ row }">
            <el-avatar :size="50" :src="row.avatar" v-if="row.avatar" />
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="用户名"
          prop="username" />
        <el-table-column
          align="center"
          label="昵称"
          prop="nickname" />
        <el-table-column
          align="center"
          label="手机号"
          prop="mobile" />
        <el-table-column
          align="center"
          label="电子邮箱"
          prop="email" />
        <el-table-column
          align="center"
          label="性别"
          prop="sex">
          <template v-slot="{row}">
            {{dictionaryMap[row.sex]}}
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="部门"
          prop="department_name">
          <template v-slot="{ row }">
            {{row.department_name || '-'}}
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="角色"
          prop="email">
          <template v-slot="{ row }">
            <el-tag v-for="item in row.roles" :key="item.id">{{item.name}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="是否启用"
          prop="status"
          width="100">
          <template v-slot="{ row }">
            <el-switch
              v-permission="'global:administrator:status'"
              @change="statusHandle(row)"
              v-model="row.status"
              :active-value="1"
              :inactive-value="0"
              :disabled="row.id === administratorId" />
          </template>
        </el-table-column>
        <el-table-column
          align="center"
          label="创建时间"
          prop="created_at"
          width="160" />
        <el-table-column
          align="center"
          label="更新时间"
          prop="updated_at"
          width="160" />
        <el-table-column
          align="center"
          label="操作"
          width="110"
          fixed="right">
          <template v-slot="{ row }">
            <el-button
              v-permission="'global:administrator:update'"
              type="primary"
              link
              @click="addEditHandle(row.id)">编辑</el-button>
            <el-button
              v-permission="'global:administrator:delete'"
              type="danger"
              link
              @click="deleteHandle(row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <AddEdit ref="refAddEdit" v-if="visible" @refresh="getList" />
    </template>
    <template #footer>
      <Page :page="page" @change="pageChangeHandle" />
    </template>
  </ContainerSidebar>
</template>

<script >
import { computed, defineComponent, reactive, ref, toRefs, nextTick, onBeforeMount } from 'vue'
import { useStore } from 'vuex'

import { ElMessage, ElMessageBox } from 'element-plus'
import ContainerSidebar from '@/components/container-sidebar/index.vue'
import EnterpriseSidebar from '@/components/enterprise-sidebar/index.vue'
import AddEdit from './components/add-edit.vue'

import usePage from '@/mixins/page'
import useDictionary from '@/mixins/dictionary'
import { clearJson, parseDate2Str } from '@/utils'

import { globalPageApi, globalDeleteApi, globalSetStatusApi } from '@/api/administrator'

export default defineComponent({
  components: { ContainerSidebar, EnterpriseSidebar, AddEdit },
  setup() {
    const store = useStore()

    const refContainerSidebar = ref()
    const refForm = ref()
    const refTable = ref()
    const refAddEdit = ref()

    const { page } = usePage()
    const { dictionaryMap, getDictionary } = useDictionary()
    const data = reactive({
      active: '',
      loading: false,
      visible: false,
      form: {
        name: '',
        department: '',
        date: []
      },
      list: [],
      selection: []
    })

    const administratorId = computed(() => store.state.administrator.administrator.id)

    const getList = () => {
      if (data.active) {
        const params = {
          id: data.active,
          name: data.form.name,
          department: data.form.department,
          start: data.form.date && data.form.date.length > 0 ? parseDate2Str(data.form.date[0]) : '',
          end: data.form.date && data.form.date.length > 1 ? parseDate2Str(data.form.date[1]) : '',
          current: page.current,
          size: page.size
        }
        data.loading = true
        globalPageApi(params).then(r => {
          if (r) {
            data.list = r.data.list,
            page.total = r.data.total
          }
          nextTick(() => { data.loading = false })
        })
      }
    }

    const reacquireHandle = () => {
      page.current = 1
      getList()
    }

    const addEditHandle = (id) => {
      data.visible = true
      nextTick(() => {
        refAddEdit.value.init(data.active, id)
      })
    }

    const deleteHandle = (id) => {
      const ids = id ? [id] : data.selection.map(item => item.id)
      ElMessageBox.confirm(`确定对[id=${ ids.join(',') }]进行[${ id ? '删除' : '批量删除' }]操作?`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        globalDeleteApi({ keys: ids }).then(r => {
          if (r) {
            ElMessage({
              message: '操作成功!',
              type: 'success'
            })
            getList()
          }
        })
      }).catch(() => {
        // to do something on canceled
      })
    }

    const statusHandle = (row) => {
      const params = {
        key: row.id,
        value: row.status
      }
      globalSetStatusApi(params).then(r => {
        if (r) {
          ElMessage({
            message: '操作成功!',
            type: 'success'
          })
        } else {
          row.status = row.status === 1 ? 0 : 1
        }
      })
    }

    const selectionHandle = (val) => {
      data.selection = val
    }

    const pageChangeHandle = (argPage) => {
      page.current = argPage.current
      page.size = argPage.size
      getList()
    }

    const changeHandle = (_row) => {
      refContainerSidebar.value.setScrollTop()
      reacquireHandle()
    }

    onBeforeMount(() => {
      getDictionary('sex')
    })

    return {
      refContainerSidebar,
      refForm,
      refTable,
      refAddEdit,
      page,
      dictionaryMap,
      ...toRefs(data),
      administratorId,
      getList,
      reacquireHandle,
      addEditHandle,
      deleteHandle,
      statusHandle,
      selectionHandle,
      pageChangeHandle,
      changeHandle,
      clearJson
    }
  }
})
</script>

<style lang="scss" scoped>

</style>
