<template>
  <el-dialog
    :title="titleName"
    :close-on-click-modal="false"
    :visible.sync="dialogVisible"
    :before-close="onClose">
    <div>
      <form-create v-model="fApi" :rule="rule" :option="options" :value.sync="value" />
      <el-row :gutter="24">
        <el-col :span="19"><div><p></p>
        </div></el-col>
      </el-row>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="dataFormSubmit()">生成订单</el-button>
      <el-button type="warning" plain aligen="center" @click="onReset">重置</el-button>
      <el-button @click="onClose">取消</el-button>
    </span>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: auto;">
      <el-table-column prop="drugId" header-align="center" align="center" width="80" label="药品编号"/>
      <el-table-column prop="drugName" header-align="center" align="center" label="药品名称"/>
      <el-table-column prop="drugDosageForm" header-align="center" align="center" label="剂型"/>
      <el-table-column prop="usageDosage" header-align="center" align="center" label="用法用量"/>
      <el-table-column prop="description" header-align="center" align="center" label="药品描述"/>
      <el-table-column prop="inventory" header-align="center" align="center" label="药品数量">
        <template slot-scope="scope">
          <el-tag size="small">X{{scope.row.inventory}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="price" header-align="center" align="center" width="100" label="单价">
        <template slot-scope="scope">
          <el-tag size="small">{{scope.row.price}}￥</el-tag>
        </template>
      </el-table-column>
    </el-table>
    <div>
      <el-col :span="24" style="margin: 0 0 20px 0">
        <el-card shadow="always">
          <h3>总价：{{this.sum}}￥</h3>
        </el-card>
      </el-col>
    </div>
  </el-dialog>
</template>

<script>
export default {
  components: {
  },
  data () {
    return {
      dialogVisible: false,
      dataListLoading: false,
      dataList: [],
      sum: '',
      fApi: {},
      isCreate: null,
      rowId: '',
      value: {},
      initValue: {},
      titleName: '',
      flag: true,
      rule: [
        {
          type: 'input',
          field: 'pharmacist',
          title: '药剂师',
          effect: {
            required: '请填写药剂师'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'estimator',
          title: '计价员',
          effect: {
            required: '请填写计价员'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'prescriptionId',
          title: '药物单编号',
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'hospital',
          title: '医院名称',
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'patientId',
          title: '患者编号',
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'deptId',
          title: '科室编号',
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'docName',
          title: '医生',
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'datePicker',
          field: 'prescriptionTime',
          title: '药物单日期',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        }
      ],
      options: {
        submitBtn: false,
        resetBtn: false
      }
    }
  },
  watch: {
    isCreate () {
      if (this.isCreate) this.titleName = '新增'
      else this.titleName = '计价'
    }
  },
  methods: {
    open (isCreate, rowId) {
      this.isCreate = isCreate
      this.rowId = rowId
      this.dialogVisible = true
      this.init()
    },
    init () {
      if (!this.isCreate) {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl(`/sys/ywd/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
            this.dataList = data.drugList
            this.getSum(data.drugList)
          } else {
            this.$message({ message: data.msg })
          }
          this.dataListLoading = false
        })
      }
    },
    // 取消
    onClose () {
      this.value = {}
      this.initValue = {}
      this.dialogVisible = false
      this.fApi.resetFields()
      this.$emit('closeHide', false)
      this.$message({ message: '取消编辑' })
    },
    // 重置
    onReset () {
      this.value = this.initValue
    },
    // 表单提交
    dataFormSubmit () {
      this.value['sum'] = this.sum
      console.log(this.value)
      this.$http({
        url: this.$http.adornUrl(`/sys/ywd/update`),
        method: 'post',
        data: this.$http.adornData(this.value)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.value = null
          this.initValue = null
          this.dialogVisible = false
          this.fApi.resetFields()
          this.$emit('closeHide')
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    getSum (drugList) {
      let sum = 0
      for (let i = 0; i < drugList.length; i++) {
        let num = drugList[i]['inventory']
        let price = drugList[i]['price']
        sum = sum + (num * price)
      }
      sum = sum.toFixed(2)
      this.sum = sum
    }
  }
}
</script>
