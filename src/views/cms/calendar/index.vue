<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">节日</label>
        <el-input
          v-model="query.name"
          class="filter-item"
          clearable
          placeholder="模糊搜索"
          style="width: 185px;"
          @keyup.enter.native="crud.toQuery"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog
        :before-close="crud.cancelCU"
        :close-on-click-modal="false"
        :title="crud.status.title"
        :visible.sync="crud.status.cu > 0"
        width="800px"
      >
        <el-form ref="form" :model="form" :rules="rules" label-width="100px" size="small">
          <el-form-item label="节日" prop="name">
            <el-input v-model="form.name" style="width: 650px;" />
          </el-form-item>
          <el-form-item label="描述" prop="description">
            <el-input v-model="form.description" style="width: 650px;" type="textarea" />
          </el-form-item>
          <el-form-item v-for="(item, index) in form.items" :key="index" :label="'休假/调课' + (index + 1)">
            <el-col>
              <el-date-picker
                v-model="item.day"
                placeholder="选择日期"
                type="date"
                value-format="yyyy-MM-dd"
              />
              <el-select v-model="item.type" placeholder="休假/调课">
                <el-option
                  v-for="option in options"
                  :key="option.value"
                  :label="option.label"
                  :value="option.value"
                />
              </el-select>
              <el-button v-if="form.items.length > 1" @click.prevent="delItem(index)">删除</el-button>
            </el-col>
          </el-form-item>
          <el-button style="width: 650px; margin-left: 100px" type="primary" @click.prevent="addItem">添加</el-button>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table
        ref="table"
        v-loading="crud.loading"
        :data="crud.data"
        size="small"
        style="width: 100%;"
        @selection-change="crud.selectionChangeHandler"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column label="节日" prop="name" />
        <el-table-column label="描述" prop="description" />
        <el-table-column
          v-if="checkPer(['admin','calendar:edit','calendar:del'])"
          align="center"
          label="操作"
          width="150px"
        >
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
import crudCalendar from '@/api/cms/calendar'
import CRUD, { crud, form, header, presenter } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import { Notification } from 'element-ui'

const defaultForm = { id: null, description: null, name: null, items: [] }
export default {
  name: 'Calendar',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '校历', url: 'api/calendar', idField: 'id', sort: 'id,desc', crudMethod: { ...crudCalendar }})
  },
  data() {
    return {
      options: [{
        value: -1,
        label: '不显示'
      }, {
        value: 0,
        label: '无'
      }, {
        value: 1,
        label: '休假'
      }, {
        value: 2,
        label: '调课'
      }],
      permission: {
        add: ['admin', 'calendar:add'],
        edit: ['admin', 'calendar:edit'],
        del: ['admin', 'calendar:del']
      },
      rules: {
        description: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'name', display_name: 'name' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    },
    [CRUD.HOOK.beforeSubmit]() {
      for (const item of this.form.items) {
        if (!item.day) {
          Notification.error({
            title: '日期未选择',
            duration: 2000
          })
          return false
        }
      }
      return true
    },
    addItem() {
      this.form.items.push({ day: '', type: 0 })
    },
    delItem(index) {
      this.form.items.splice(index, 1)
    }
  }
}
</script>

<style scoped>

</style>
