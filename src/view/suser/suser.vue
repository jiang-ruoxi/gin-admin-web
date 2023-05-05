<template>
  <div>
    <div class="gva-search-box">
      <el-form :inline="true" :model="searchInfo" class="demo-form-inline" @keyup.enter="onSubmit">
        <el-form-item label="创建时间">
          <el-date-picker v-model="searchInfo.startCreatedAt" type="datetime" placeholder="开始时间"></el-date-picker>
          —
          <el-date-picker v-model="searchInfo.endCreatedAt" type="datetime" placeholder="结束时间"></el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="search" @click="onSubmit">查询</el-button>
          <el-button icon="refresh" @click="onReset">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="gva-table-box">
      <div class="gva-btn-list">
        <el-button type="primary" icon="plus" @click="openDialog">新增</el-button>
      </div>
      <el-table
          ref="multipleTable"
          style="width: 100%"
          tooltip-effect="dark"
          :data="tableData"
          row-key="ID"
          @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55"/>
        <el-table-column align="left" label="编号" prop="id" width="100"/>
        <el-table-column align="left" label="OpenId" prop="openId" width="300"/>
        <el-table-column align="left" label="用户昵称" prop="nickName" width="150"/>
        <el-table-column prop="headUrl" label="用户头像" min-width="200">
          <template #default="scope">
            <img :src="scope.row.headUrl" min-width="80" height="80"/>
          </template>
        </el-table-column>
        <el-table-column align="left" label="所属地区" prop="area" width="150"/>
        <el-table-column align="left" label="注册时间" prop="addTime" width="200"/>
        <el-table-column align="left" label="操作" width="300">
          <template #default="scope">
            <el-button type="primary" link icon="edit" class="table-button" @click="updateUserFunc(scope.row)">修改
            </el-button>
            <el-button type="primary" link icon="delete" @click="deleteRow(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="gva-pagination">
        <el-pagination
            layout="total, sizes, prev, pager, next, jumper"
            :current-page="page"
            :page-size="pageSize"
            :page-sizes="[10, 30, 50, 100]"
            :total="total"
            @current-change="handleCurrentChange"
            @size-change="handleSizeChange"
        />
      </div>
    </div>
    <el-dialog v-model="dialogFormVisible" :before-close="closeDialog" title="弹窗操作">
      <el-form :model="formData" label-position="right" ref="elFormRef" :rules="rule" label-width="80px">
        <el-form-item label="OpenId:" prop="openId">
          <el-input v-model="formData.openId" :clearable="true" placeholder="请输入"/>
        </el-form-item>
        <el-form-item label="用户昵称:" prop="nickName">
          <el-input v-model="formData.nickName" :clearable="true" placeholder="请输入"/>
        </el-form-item>
        <el-form-item label="头像:" prop="headUrl">
          <el-input v-model="formData.headUrl" :clearable="true" placeholder="请输入"/>
        </el-form-item>
        <el-form-item label="地区:" prop="area">
          <el-input v-model="formData.area" :clearable="true" placeholder="请输入"/>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="closeDialog">取 消</el-button>
          <el-button type="primary" @click="enterDialog">确 定</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'User'
}
</script>

<script setup>
import {
  getUserList,
  findUser,
  deleteSUser,
  createUser,
  updateUser,
} from '@/api/suser'

// 全量引入格式化工具 请按需保留
import {getDictFunc, formatDate, formatBoolean, filterDict} from '@/utils/format'
import {ElMessage, ElMessageBox} from 'element-plus'
import {ref, reactive} from 'vue'


// 自动化生成的字典（可能为空）以及字段
const formData = ref({
  openId: "",
})

// 验证规则
const rule = reactive({
  openId: [{
    required: true,
    message: '',
    trigger: ['input', 'blur'],
  }],
  nickName: [{
    required: true,
    message: '',
    trigger: ['input', 'blur'],
  }],
  headUrl: [{
    required: true,
    message: '',
    trigger: ['input', 'blur'],
  }],
})

const elFormRef = ref()


// =========== 表格控制部分 ===========
const page = ref(1)
const total = ref(0)
const pageSize = ref(10)
const tableData = ref([])
const searchInfo = ref({})

// 重置
const onReset = () => {
  searchInfo.value = {}
  getTableData()
}

// 搜索
const onSubmit = () => {
  page.value = 1
  pageSize.value = 10
  getTableData()
}

// 分页
const handleSizeChange = (val) => {
  pageSize.value = val
  getTableData()
}

// 修改页面容量
const handleCurrentChange = (val) => {
  page.value = val
  getTableData()
}

// 行为控制标记（弹窗内部需要增还是改）
const type = ref('')
// 更新行
const updateUserFunc = async (row) => {
  console.log(row)
  const res = await findUser({id: row.id})
  type.value = 'update'
  if (res.code === 0) {
    formData.value = res.data.rebaike
    dialogFormVisible.value = true
  }
}

// 删除行
const deleteRow = (row) => {
  ElMessageBox.confirm('确定要删除吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    deleteUserFunc(row)
  })
}

// 删除行
const deleteUserFunc = async (row) => {
  const res = await deleteSUser({id: row.id})
  if (res.code === 0) {
    ElMessage({
      type: 'success',
      message: '删除成功'
    })
    if (tableData.value.length === 1 && page.value > 1) {
      page.value--
    }
    getTableData()
  }
}

// 弹窗控制标记
const dialogFormVisible = ref(false)

// 打开弹窗
const openDialog = () => {
  type.value = 'create'
  dialogFormVisible.value = true
}

// 关闭弹窗
const closeDialog = () => {
  dialogFormVisible.value = false
  formData.value = {

  }
}
// 弹窗确定
const enterDialog = async () => {
  elFormRef.value?.validate(async (valid) => {
    if (!valid) return
    let res
    switch (type.value) {
      case 'create':
        res = await createUser(formData.value)
        break
      case 'update':
        res = await updateUser(formData.value)
        break
      default:
        res = await createUser(formData.value)
        break
    }
    if (res.code === 0) {
      ElMessage({
        type: 'success',
        message: '创建/更改成功'
      })
      closeDialog()
      getTableData()
    }
  })
}

// 查询
const getTableData = async () => {
  const table = await getUserList({page: page.value, pageSize: pageSize.value, ...searchInfo.value})
  if (table.code === 0) {
    tableData.value = table.data.list
    total.value = table.data.total
    page.value = table.data.page
    pageSize.value = table.data.pageSize
  }
}

getTableData()
</script>

<style>
</style>
