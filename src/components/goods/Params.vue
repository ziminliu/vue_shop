<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        title="只允许为第三级分类"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col
          ><span>选择商品分类：</span
          ><el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="handleChange"
          ></el-cascader
        ></el-col>
      </el-row>

      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="用户管理" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border="" stripe="">
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag 标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable=""
                  @close="handleClose(i, scope.row)"
                  >{{ item }}</el-tag
                >
                <!-- 添加tag 标签 -->
                <el-input
                  class="input-new-tag"
                  v-show="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-show="!scope.row.inputVisible"
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope"
                ><el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="配置管理" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
            >添加属性</el-button
          >
          <!-- 静态参数表格 -->
          <el-table :data="onlyTableData" border="" stripe="">
            <!-- 展开行 --><!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag 标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable=""
                  @close="handleClose(i, scope.row)"
                  >{{ item }}</el-tag
                >
                <!-- 添加tag 标签 -->
                <el-input
                  class="input-new-tag"
                  v-show="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-show="!scope.row.inputVisible"
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope"
                ><el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      :close="addDialogClosed"
    >
      <!-- 添加参数的表格 -->
      <el-form
        ref="addFormRef"
        :model="addForm"
        label-width="100px"
        :rules="addFormRules"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogClosed">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      :close="editDialogClosed"
    >
      <!-- 添加参数的表格 -->
      <el-form
        ref="editFormRef"
        :model="editForm"
        label-width="100px"
        :rules="editFormRules"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogClosed">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类信息
      catelist: [],
      // 级联选择器的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 级联选择器选择的数组内容
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数的数据
      manyTableData: [],
      // 静态参数的数据
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 表单数据对象
      addForm: {},
      // 表单验证规则对象
      addFormRules: {
        attr_name: [{ required: true, message: '请输入参数', trigger: 'blur' }]
      },
      // 修改对话框显示与否
      editDialogVisible: false,
      // 修改的表单数据对象
      editForm: {},
      // 修改表单的表单验证规则对象
      editFormRules: {
        attr_name: [{ required: true, message: '请输入参数', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
    console.log(this.selectedCateKeys.length)
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catelist = res.data
      // console.log(this.catelist)
    },
    // 级联选择器的改变触发事件
    handleChange() {
      this.getParamsData()
    },
    // tabs 页签点击事件的处理函数
    handleTabClick() {
      // console.log(this.activeName)
      this.getParamsData()
    },
    // 发起请求获取服务器数据
    async getParamsData() {
      // 为选中三级菜单则重置
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 选中3级 分类
      // console.log(this.selectedCateKeys)
      // 发起请求
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取参数失败')
      }
      console.log(res.data)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 默认每一行各自的输入文本框都隐藏
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 关闭添加属性对话框的事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
      this.addDialogVisible = false
    },
    // 点击按钮，添加参数
    async addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！！')
        }

        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 显示编辑对话框
    async showEditDialog(row) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败')
      }
      console.log(res.data, 'res.data')
      // console.log(row)
      this.editForm = row
      this.editDialogVisible = true
      // console.log(row, 'row')
      // console.log(this.editForm, 'editform')
    },
    // 关闭触发的事件
    editDialogClosed() {
      // console.log(this.editForm, 'editForm')
      this.$refs.editFormRef.resetFields()
      this.editDialogVisible = false
    },
    // 点击按钮，修改参数信息
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // console.log(this.lostData)
        const { data: res } = await this.$http
          .put(
            `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
            {
              attr_name: this.editForm.attr_name,
              attr_sel: this.activeName
            }
          )
          .then(this.saveAttrVals(this.editForm))
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        // console.log(res.data)
        this.$message.success('修改参数成功')
        this.getParamsData()
        this.editDialogClosed()
      })
    },
    // 根据id 删除对应的参数项
    async removeParams(row) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 用户取消删除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      // 删除的业务操作
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${row.attr_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除属性失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或按下回车
    handleInputConfirm(row) {
      // 回车触发两次
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        console.log('回车引发的二次触发')
        row.inputVisible = false
        return
      }
      // // 前端修改
      console.log(row.inputValue)
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发起请求保存
      this.saveAttrVals(row)
    },
    // 点击按钮展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // 页面重新渲染后，执行回调函数
      // console.log(this.$refs)
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除标签操作
    async handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      const res = await this.saveAttrVals(row)
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数失败！')
      }
      this.$message.success('修改参数成功')
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      return res
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分裂的ID
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin-top: 15px;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
