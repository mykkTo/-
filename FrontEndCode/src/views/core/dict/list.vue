<template>
  <div class="app-container">
    <div style="margin-bottom: 10px">
      <el-button
        type="primary"
        size="mini"
        icon="el-icon-download"
        @click="dialogVisible = true"
      >
        导入Excel
      </el-button>
      <el-button
        type="primary"
        size="mini"
        icon="el-icon-upload2"
        @click="exportData"
      >
        导出Excel
      </el-button>
    </div>

    <el-table :data="list" border row-key="id" lazy :load="load">
      <el-table-column label="名称" align="left" prop="name" />
      <el-table-column label="编码" prop="dictCode" />
      <el-table-column label="值" align="left" prop="value" />
    </el-table>

    <el-dialog title="数据字典导入" :visible.sync="dialogVisible" width="30%">
      <el-form>
        <el-form-item label="请选择Excel文件">
          <el-upload
            :auto-upload="true"
            :multiple="false"
            :limit="1"
            :on-exceed="fileUploadExceed"
            :on-success="fileUploadSuccess"
            :on-error="fileUploadError"
            :action="BASE_API + '/admin/core/dict/import'"
            name="file"
            accept="application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
          >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import dictApi from '@/api/core/dict'
export default {
  // 定义数据
  data() {
    return {
      dialogVisible: false, // 文件上传对话框是否显示
      BASE_API: process.env.VUE_APP_BASE_API, // 获取后端接口地址
      list: [], // 数据字典列表
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    // 调用api层获取数据库中的数据
    fetchData() {
      dictApi.listByParentId(1).then((response) => {
        this.list = response.data.list
      })
    },

    // 延迟加载子节点
    getChildren(row, treeNode, resolve) {
      dictApi.listByParentId(row.id).then((response) => {
        // 负责将子节点数据展示在展开的列表中
        resolve(response.data.list)
      })
    },
    // Excel数据导出
    exportData() {
      window.location.href = this.BASE_API + '/admin/core/dict/export'
    },
    // 上传多于一个文件时
    fileUploadExceed() {
      this.$message.warning('只能选取一个文件')
    },

    // 上传成功回调
    fileUploadSuccess(response) {
      if (response.code === 0) {
        this.$message.success('数据导入成功')
        this.dialogVisible = false
        this.fetchData()
      } else {
        this.$message.error(response.message)
      }
    },

    // 上传失败回调
    // eslint-disable-next-line handle-callback-err
    fileUploadError(error) {
      this.$message.error('数据导入失败')
    },
  },
}
</script>
