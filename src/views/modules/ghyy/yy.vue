<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.patientName" placeholder="患者名" clearable @change="getDataList"></el-input>
      </el-form-item>

      <el-form-item>
        <el-input v-model="dataForm.deptName" placeholder="科室名称" clearable @change="getDataList"></el-input>
      </el-form-item>

      <el-form-item>
        <el-input v-model="dataForm.name" placeholder="医生名称" clearable @change="getDataList"></el-input>
      </el-form-item>

      <el-select v-model="dataForm.status" placeholder="是否过期" clearable @change="getDataList">
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
      <el-table-column prop="bookingId" header-align="center" align="center" width="80" label="预约ID"/>
      <el-table-column prop="patientName" header-align="center" align="center" label="患者名"/>
      <el-table-column prop="deptName" header-align="center" align="center" label="预约科室"/>
      <el-table-column prop="name" header-align="center" align="center" label="预约医生"/>
      <el-table-column prop="sex" header-align="center" align="center" width="50" label="性别">
        <template slot-scope="scope">
          <span style="margin-right: 10px;">{{ scope.row.sex === '0' ? '女' : '男' }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="age" header-align="center" align="center" label="年龄"/>
      <el-table-column prop="mobile" header-align="center" align="center" label="手机号"/>
      <el-table-column prop="bookingTime" header-align="center" align="center" width="180" label="预约时间"/>
      <el-table-column prop="status" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">已就诊</el-tag>
          <el-tag v-if="scope.row.status === 2" size="small" type="info">已过期</el-tag>
          <el-tag v-if="scope.row.status === 1" size="small">未就诊</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.status !== 0 && scope.row.status !== 2" type="text" size="small" @click="updateHandle(scope.row.bookingId)">已就诊</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.bookingId)">删除</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './gh-temp'
export default {
  data () {
    return {
      dataForm: {
        patientName: '',
        deptName: '',
        name: '',
        selectDate: [],
        starDate: '',
        endDate: '',
        status: ''
      },
      options: [{value: '1', label: '未就诊'}, {value: '0', label: '已就诊'}, {value: '2', label: '已过期'}],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  created () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/sys/yy/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'patientName': this.dataForm.patientName,
          'deptName': this.dataForm.deptName,
          'name': this.dataForm.name,
          'starDate': this.dataForm.starDate,
          'endDate': this.dataForm.endDate,
          'status': this.dataForm.status
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.list
          this.totalPage = data.list.length / this.pageSize * 10
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
    // 新增
    addOrUpdateHandle (patientId, patientName) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(patientId, patientName)
      })
    },
    // 修改
    updateHandle (bookingId) {
      this.$confirm(`确定该用户已就诊?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/yy/status/' + bookingId),
          method: 'get',
          data: this.$http.adornParams()
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
      this.getDataList()
    },
    // 删除
    deleteHandle (id) {
      var bookingIds = id ? [id] : this.dataListSelections.map(item => {
        return item.bookingId
      })
      this.$confirm(`确定对该预约信息进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/yy/delete'),
          method: 'post',
          data: this.$http.adornData(bookingIds, false)
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
