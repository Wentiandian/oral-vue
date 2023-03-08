<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.eleRecordsId" placeholder="病历编号" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.patientName" placeholder="患者名" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-select v-model="dataForm.deptId" placeholder="治疗科室" clearable @change="getDataList">
        <el-option v-for="item in deptList" :key="item.value" :label="item.label" :value="item.value" @change="getDataList">
        </el-option>
      </el-select>
      <el-select v-model="dataForm.isReferral" placeholder="是否复诊" clearable @change="getDataList">
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
        <el-button type="primary" @click="add()">新增病历</el-button>
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
      <el-table-column prop="eleRecordsId" header-align="center" align="center" label="病历编号"/>
      <el-table-column prop="drugId" header-align="center" align="center" label="药物单(处方)编号"/>
      <el-table-column prop="patientName" header-align="center" align="center" label="患者名"/>
      <el-table-column prop="deptName" header-align="center" align="center" label="治疗科室"/>
      <el-table-column prop="treatmentNum" header-align="center" align="center" label="第几次就诊">
        <template slot-scope="scope">
          <span style="margin-right: 10px;">第{{ scope.row.treatmentNum }}次就诊</span>
        </template>
      </el-table-column>
      <el-table-column prop="treatmentTime" header-align="center" align="center" width="180" label="就诊时间"/>
      <el-table-column prop="isReferral" header-align="center" align="center" label="是否复诊">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isReferral === 0" size="small" type="danger">否</el-tag>
          <el-tag v-if="scope.row.isReferral === 1" size="small">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="edit(scope.row.eleRecordsId)">详细</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.eleRecordsId)">删除</el-button>
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
import infoTemp from './blgl-temp'
export default {
  components: {
    infoTemp
  },
  data () {
    return {
      dataForm: {
        eleRecordsId: '',
        patientName: '',
        deptId: '',
        isReferral: '',
        selectDate: [],
        starDate: '',
        endDate: ''
      },
      options: [{value: '1', label: '是'}, {value: '0', label: '否'}],
      dataList: [],
      deptList: [],
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
    this.getDeptList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/sys/dzbl/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'eleRecordsId': this.dataForm.eleRecordsId,
          'patientName': this.dataForm.patientName,
          'deptId': this.dataForm.deptId,
          'isReferral': this.dataForm.isReferral,
          'starDate': this.dataForm.starDate,
          'endDate': this.dataForm.endDate
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.list.records
          this.totalPage = data.list.pages
          this.getDeptName(this.dataList)
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    getDeptList () {
      this.$http({
        url: this.$http.adornUrl('/sys/common/deptList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let deptMap = {}
          for (let i = 0; i < data.list.length; i++) {
            deptMap = {
              'label': data.list[i].deptName,
              'value': data.list[i].deptId
            }
            this.deptList.push(deptMap)
          }
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    getDeptName (dataList) {
      for (let i = 0; i < dataList.length; i++) {
        for (let j = 0; j < this.deptList.length; j++) {
          if (this.deptList[j]['value'] === dataList[i].deptId) {
            dataList[i]['deptName'] = this.deptList[j]['label']
          }
        }
      }
      this.dataList = dataList
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
    // 新增
    add () {
      // 使用目标页面的方法
      this.$refs.infoTemp.open(true, '')
    },
    // 编辑
    edit (id) {
      this.$refs.infoTemp.open(false, id)
    },
    // 删除
    deleteHandle (id) {
      var eleRecordsIds = id ? [id] : this.dataListSelections.map(item => {
        return item.eleRecordsId
      })
      this.$confirm(`确定对该挂号信息进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/dzbl/delete'),
          method: 'post',
          data: this.$http.adornData(eleRecordsIds, false)
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
