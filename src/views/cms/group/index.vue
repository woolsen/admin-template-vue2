<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">群号</label>
        <el-input v-model="query.groupId" clearable placeholder="群号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">群名</label>
        <el-input v-model="query.groupName" clearable placeholder="群名" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="群号" prop="groupId">
            <el-input v-model="form.groupId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="群名" prop="groupName">
            <el-input v-model="form.groupName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="排序值" prop="sortNum">
            <el-input v-model="form.sortNum" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="groupId" label="群号" />
        <el-table-column prop="groupName" label="群名" />
        <el-table-column prop="sortNum" label="排序值" />
        <el-table-column v-if="checkPer(['admin','group:edit','group:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudGroup from '@/api/cms/group'
import CRUD, { crud, form, header, presenter } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, groupId: null, groupName: null, sortNum: null }
export default {
  name: 'Group',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '群管理', url: 'api/group', idField: 'id', sort: 'sortNum,asc', crudMethod: { ...crudGroup }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'group:add'],
        edit: ['admin', 'group:edit'],
        del: ['admin', 'group:del']
      },
      rules: {
        groupId: [
          { required: true, message: '群号不能为空', trigger: 'blur' }
        ],
        groupName: [
          { required: true, message: '群名不能为空', trigger: 'blur' }
        ],
        sortNum: [
          { required: true, message: '排序值不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'groupId', display_name: '群号' },
        { key: 'groupName', display_name: '群名' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
