<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.drugId" placeholder="药品编号" clearable @change="getDataList"></el-input>
      </el-form-item>
      <el-select v-model="dataForm.drugName" placeholder="药品名称" clearable @change="getDataList">
        <el-option v-for="item in drugNameList" :key="item.value" :label="item.label" :value="item.value" @change="getDataList">
        </el-option>
      </el-select>
      <el-select v-model="dataForm.drugDosageForm" placeholder="药品剂型" clearable @change="getDataList">
        <el-option v-for="item in drugDosageList" :key="item.value" :label="item.label" :value="item.value" @change="getDataList">
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
        <el-button type="primary" @click="add()">新增药品</el-button>
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
      <el-table-column prop="drugId" header-align="center" align="center" width="80" label="药品编号"/>
      <el-table-column prop="drugName" header-align="center" align="center" label="药品名称"/>
      <el-table-column prop="imageName" label="药品图片" align="center">
        <template slot-scope="{ row }">
          <div slot="error" class="image-slot" v-if="row.imageName !== null">
            <img :src="getImage(row.imageName)" style="width: auto; height: 55px; border:none;" >
          </div>
          <div slot="error" class="image-slot" v-else>
            <img src="src/assets/img/logo/logo16.png" style="width: auto; height: 55px; border:none;" >
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="drugDosageForm" header-align="center" align="center" label="药品剂型"/>
      <el-table-column prop="drugSize" header-align="center" align="center" label="药品规格"/>
      <el-table-column prop="price" header-align="center" align="center" label="单价"/>
      <el-table-column prop="inventory" header-align="center" align="center" label="库存量"/>
      <el-table-column prop="usageDosage" header-align="center" align="center" label="用法用量"/>
      <el-table-column prop="address" header-align="center" align="center" label="存放地址"/>
      <el-table-column prop="lifeTime" header-align="center" align="center" label="保质期/月"/>
      <el-table-column prop="status" header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="warning">未入库</el-tag>
          <el-tag v-else size="small">已入库</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="edit(scope.row.drugId)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.drugId)">删除</el-button>
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
import infoTemp from './ypfl-temp'
export default {
  components: {
    infoTemp
  },
  data () {
    return {
      dataForm: {
        drugId: '',
        drugName: '',
        drugDosageForm: '',
        selectDate: [],
        starDate: '',
        endDate: ''
      },
      drugNameList: [],
      drugDosageList: [],
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
    this.getDrugDosageList()
    this.getDrugNameList()
  },
  methods: {
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/sys/ypfl/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'drugId': this.dataForm.drugId,
          'drugName': this.dataForm.drugName,
          'drugDosageForm': this.dataForm.drugDosageForm,
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
    getDrugDosageList () {
      this.$http({
        url: this.$http.adornUrl('/sys/common/drugDosageList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let drugDosageMap = {}
          for (let i = 0; i < data.list.length; i++) {
            drugDosageMap = {
              'label': data.list[i].drugDosageForm,
              'value': data.list[i].drugDosageForm
            }
            this.drugDosageList.push(drugDosageMap)
          }
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    getDrugNameList () {
      this.$http({
        url: this.$http.adornUrl('/sys/common/drugNameList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let drugDosageMap = {}
          for (let i = 0; i < data.list.length; i++) {
            drugDosageMap = {
              'label': data.list[i].drugName,
              'value': data.list[i].drugName
            }
            this.drugNameList.push(drugDosageMap)
          }
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    getImage (imageName) {
      return `http://localhost:8080/sys/common/download?name=${imageName}`
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
      var drugIds = id ? [id] : this.dataListSelections.map(item => {
        return item.drugId
      })
      this.$confirm(`确定对该科室信息进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/ypfl/delete'),
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
