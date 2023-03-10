<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.eleRecordsId" placeholder="病历编号" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="add()">上传文件</el-button>
      </el-form-item>
    </el-form>
    <el-row :gutter="12">
      <el-col :span="6" v-for="item in this.value" :key="item.fileId">
          <div @click="edit(item.fileId, item.relationId)">
            <el-card shadow="hover">
              <i class="el-icon-document" style="font-size: 80px;margin-left: 80px"></i>
              <p style="margin-left: 50px">{{item.fileName}}</p>
            </el-card>
          </div>
        <el-button type="danger" size="small" style="margin-left: 220px;margin-top:-40px;float: left" @click="deleteHandle(item.fileName)">删除</el-button>
      </el-col>
    </el-row>
    <!-- 弹窗, 新增 / 修改 -->
    <infoTemp ref="infoTemp"></infoTemp>
  </div>
</template>

<script>
import infoTemp from './blwdgl-temp'
export default {
  components: {
    infoTemp
  },
  data () {
    return {
      dataForm: {
        eleRecordsId: ''
      },
      value: {},
      dataListLoading: false
    }
  },
  created () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/sys/blwdgl/list'),
        method: 'get',
        params: this.$http.adornParams({
          'eleRecordsId': this.dataForm.eleRecordsId
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          console.log(data)
          this.value = data.list
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 新增
    add () {
      // 使用目标页面的方法
      this.$refs.infoTemp.open(true, '')
    },
    // 编辑
    edit (id, relationId) {
      this.$refs.infoTemp.open(false, id, relationId)
    },
    // 删除
    deleteHandle (fileName) {
      this.$confirm(`确定对该文件进行删除操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/blwdgl/delete'),
          method: 'post',
          data: this.$http.adornData(fileName, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {})
    }
  }
}
</script>
