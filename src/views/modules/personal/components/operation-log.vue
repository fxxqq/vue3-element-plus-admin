<template>
  <div class="operation-log-container">
    <el-timeline v-if="list.length">
      <el-timeline-item
        placement="top"
        v-for="item in list"
        :key="item.id"
        :timestamp="item.created_at">
        <el-card>
          <el-descriptions :column="2">
            <el-descriptions-item label="操作信息">{{item.operation}}</el-descriptions-item>
            <el-descriptions-item label="浏览器">{{item.browser}}</el-descriptions-item>
            <el-descriptions-item label="IP">{{item.ip}}</el-descriptions-item>
            <el-descriptions-item label="操作系统">{{item.operating_system}}</el-descriptions-item>
          </el-descriptions>
        </el-card>
      </el-timeline-item>
    </el-timeline>
    <el-empty v-else />
  </div>

</template>

<script>
import { defineComponent, reactive, nextTick, onBeforeMount, toRefs } from 'vue'

import usePage from '@/mixins/page'

import { selfPageApi } from '@/api/operation-log'

export default defineComponent({
  setup() {
    const { page } = usePage()
    const data = reactive({
      loading: false,
      list: []
    })

    const getList = () => {
      const params = {
        current: page.current,
        size: page.size
      }
      data.loading = true
      selfPageApi(params).then(r => {
        if (r) {
          data.list = r.data.list
          page.total = r.data.total
        }
        nextTick(() => { data.loading = false })
      })
    }

    onBeforeMount(() => {
      getList()
    })

    return {
      page,
      ...toRefs(data)
    }
  }
})
</script>

<style lang="scss" scoped>
.operation-log-container {
  // max-height: 600px;
}
</style>
