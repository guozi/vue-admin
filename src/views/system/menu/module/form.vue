<template>
  <el-dialog :append-to-body="true" :visible.sync="dialog" :title="isAdd ? '新增菜单' : '编辑菜单'" width="600px">
    <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
      <el-form-item label="菜单图标">
        <el-popover
          placement="bottom-start"
          width="460"
          trigger="click"
          @show="$refs['iconSelect'].reset()">
          <IconSelect ref="iconSelect" @selected="selected" />
          <el-input slot="reference" v-model="form.icon" style="width: 460px;" placeholder="点击选择图标" readonly>
            <svg-icon v-if="form.icon" slot="prefix" :icon-class="form.icon" class="el-input__icon" style="height: 32px;width: 16px;" />
            <i v-else slot="prefix" class="el-icon-search el-input__icon"/>
          </el-input>
        </el-popover>
      </el-form-item>
      <el-form-item label="菜单名称" prop="name">
        <el-input v-model="form.name" placeholder="名称" style="width: 460px;"/>
      </el-form-item>
      <el-form-item label="菜单排序" prop="num">
        <el-input v-model.number="form.num" placeholder="序号越小越靠前" style="width: 460px;"/>
      </el-form-item>
      <el-form-item label="链接地址">
        <el-input v-model="form.url" placeholder="菜单路径" style="width: 460px;"/>
      </el-form-item>
      <el-form-item label="组件路径">
        <el-input v-model="form.code" placeholder="菜单路径" style="width: 460px;"/>
      </el-form-item>
      <el-form-item label="上级类目">
        <treeselect v-model="form.parentId" :options="menus" style="width: 460px;" placeholder="选择上级类目" />
      </el-form-item>
      <el-form-item style="margin-top: -10px;margin-bottom: 0px;" label="选择角色">
        <treeselect v-model="roleIds" :multiple="true" :options="roles" style="width: 460px;" placeholder="请选择角色" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="text" @click="cancel">取消</el-button>
      <el-button :loading="loading" type="primary" @click="doSubmit">确认</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { add, edit } from '@/api/menu'
import Treeselect from '@riophae/vue-treeselect'
import IconSelect from '@/components/IconSelect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
export default {
  components: { Treeselect, IconSelect },
  props: {
    menus: {
      type: Array,
      required: true
    },
    roles: {
      type: Array,
      required: true
    },
    isAdd: {
      type: Boolean,
      required: true
    },
    sup_this: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      loading: false, dialog: false,
      form: { name: '', num: 999, url: '', code: '', iframe: 'false', roles: [], parentId: 0, icon: '' }, roleIds: [],
      rules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        num: [
          { required: true, message: '请输入序号', trigger: 'blur', type: 'number' }
        ],
        iframe: [
          { required: true, message: '请选择菜单类型', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    cancel() {
      this.resetForm()
    },
    doSubmit() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.loading = true
          this.form.roles = []
          const _this = this
          this.roleIds.forEach(function(data, index) {
            const role = { id: data }
            _this.form.roles.push(role)
          })
          if (this.isAdd) {
            this.doAdd()
          } else this.doEdit()
        } else {
          return false
        }
      })
    },
    doAdd() {
      add(this.form).then(res => {
        this.resetForm()
        this.$notify({
          title: '添加成功',
          type: 'success',
          duration: 2500
        })
        this.loading = false
        this.$parent.$parent.init()
        this.$parent.$parent.getMenus()
      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })
    },
    doEdit() {
      edit(this.form).then(res => {
        this.resetForm()
        this.$notify({
          title: '修改成功',
          type: 'success',
          duration: 2500
        })
        this.loading = false
        this.sup_this.init()
        this.sup_this.getMenus()
      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })
    },
    resetForm() {
      this.dialog = false
      this.$refs['form'].resetFields()
      this.form = { name: '', num: 999, url: '', code: '', iframe: 'false', roles: [], pid: 0, icon: '' }
      this.roleIds = []
    },
    selected(name) {
      this.form.icon = name
    }
  }
}
</script>

<style scoped>

</style>
