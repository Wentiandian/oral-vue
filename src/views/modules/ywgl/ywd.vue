<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.prescriptionId" placeholder="药物单编号" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.patientId" placeholder="患者编号" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-select v-model="dataForm.status" placeholder="状态" clearable @change="getDataList">
        <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" @change="getDataList">
        </el-option>
      </el-select>
      <el-form-item>
        <el-date-picker
          @change="dateSelect"
          v-model="dataForm.selectDate"
          type="datetimerange"
          value-format="yyyy-MM-dd"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"/>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="prescriptionId" header-align="center" align="center" width="80" label="药物单编号"/>
      <el-table-column prop="hospital" header-align="center" align="center" label="医院名称"/>
      <el-table-column prop="patientId" header-align="center" align="center" label="患者编号"/>
      <el-table-column prop="deptId" header-align="center" align="center" label="科室编号"/>
      <el-table-column prop="prescriptionTime" header-align="center" align="center" label="药物单日期"/>
      <el-table-column prop="docName" header-align="center" align="center" label="医生"/>
      <el-table-column prop="pharmacist" header-align="center" align="center" label="药剂师"/>
      <el-table-column prop="estimator" header-align="center" align="center" label="计价员"/>
      <el-table-column prop="useMethod" header-align="center" align="center" label="备注"/>
      <el-table-column prop="status" header-align="center" align="center" width="100" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === '0'" size="small" type="warning">未计价</el-tag>
          <el-tag v-else size="small">已计价</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="180" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="edit(scope.row.prescriptionId)">计价</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.prescriptionId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <infoTemp ref="infoTemp" @closeHide="getDataList"></infoTemp>
  </div>
</template>

<script>
import infoTemp from './ywd-temp'
export default {
  components: {
    infoTemp
  },
  data () {
    return {
      dataForm: {
        prescriptionId: '',
        patientId: '',
        status: '',
        selectDate: [],
        starDate: '',
        endDate: ''
      },
      options: [{value: '1', label: '已计价'}, {value: '0', label: '未计价'}],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
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
        url: this.$http.adornUrl('/sys/ywd/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'prescriptionId': this.dataForm.prescriptionId,
          'patientId': this.dataForm.patientId,
          'status': this.dataForm.status,
          'starDate': this.dataForm.starDate,
          'endDate': this.dataForm.endDate
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.list.records
          this.totalPage = data.list.pages
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    dateSelect () {
      if (this.dataForm.selectDate === null) {
        this.dataForm.starDate = ''
        this.dataForm.endDate = ''
      } else {
        this.dataForm.starDate = this.dataForm.selectDate[0]
        this.dataForm.endDate = this.dataForm.selectDate[1]
      }
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 编辑
    edit (id) {
      this.$refs.infoTemp.open(false, id)
    },
    // 删除
    deleteHandle (id) {
      var drugIds = id ? [id] : this.dataListSelections.map(item => {
        return item.drugId
      })
      this.$confirm(`确定对该科室信息进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/ypcrk/delete'),
          method: 'post',
          data: this.$http.adornData(drugIds, false)
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
