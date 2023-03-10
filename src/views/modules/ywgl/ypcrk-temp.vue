<template>
  <el-dialog
    :title="titleName"
    :close-on-click-modal="false"
    :visible.sync="dialogVisible"
    :before-close="onClose">
    <div>
      <form-create v-model="fApi" :rule="rule" :option="options" :value.sync="value" />
      <el-row :gutter="24">
        <el-col :span="19"><div><p></p></div></el-col>
      </el-row>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      <el-button type="warning" plain aligen="center" @click="onReset">重置</el-button>
      <el-button @click="onClose">取消</el-button>
    </span>
  </el-dialog>
</template>

<script>
import Upload from '@/views/common/upload'
export default {
  components: {
    Upload
  },
  data () {
    return {
      dialogVisible: false,
      fApi: {},
      isCreate: null,
      functions: '',
      rowId: '',
      value: {},
      initValue: {},
      drugNameList: [],
      titleName: '',
      flag: true,
      rule: [],
      inBoundRule: [
        {
          type: 'input',
          field: 'drugId',
          title: '药品编号',
          effect: {
            required: true
          },
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'select',
          field: 'drugName',
          title: '药品名称',
          options: [],
          effect: {
            required: '请选择药品名称'
          },
          props: {
            disabled: this.flag
          },
          on: {
            change: () => {
              this.selectDrugName(this.fApi.getValue('drugName'))
            }
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'inventory',
          title: '入库数量',
          effect: {
            required: '请填写入库数量'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'vendor',
          title: '供应商',
          effect: {
            required: '请填写供应商'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'inboundBatch',
          title: '入库批次',
          effect: {
            required: '请填写入库批次'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'inboundTime',
          title: '入库时间',
          effect: {
            required: '请选择入库时间'
          },
          col: {
            span: 12
          }
        }
      ],
      outBoundRule: [
        {
          type: 'input',
          field: 'drugId',
          title: '药品编号',
          effect: {
            required: true
          },
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'select',
          field: 'drugName',
          title: '药品名称',
          options: [],
          props: {
            disabled: true
          },
          on: {
            change: () => {
              this.selectDrugName(this.fApi.getValue('drugName'))
            }
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'inventory',
          title: '出库数量',
          effect: {
            required: '请填写出库数量'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'outboundBatch',
          title: '出库批次',
          effect: {
            required: '请填写入库批次'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'outboundTime',
          title: '出库时间',
          effect: {
            required: '请选择出库时间'
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
      this.rule = this.inBoundRule
      if (this.isCreate) this.titleName = '新增'
      else this.titleName = '编辑'
    }
  },
  methods: {
    open (isCreate, rowId, functions) {
      this.functions = functions
      this.isCreate = isCreate
      this.rowId = rowId
      this.dialogVisible = true
      if (isCreate) {
        this.flag = false
      } else {
        this.flag = true
      }
      console.log(this.flag)
      this.init()
      this.getDrugNameList()
    },
    outBoundInfo (isCreate, rowId, functions) {
      this.functions = functions
      this.isCreate = isCreate
      this.rowId = rowId
      this.rule = this.outBoundRule
      this.flag = true
      this.titleName = '出库'
      this.functions = '2'
      this.dialogVisible = true
      this.init()
      this.getDrugNameList()
    },
    init () {
      if (!this.isCreate) {
        this.$http({
          url: this.$http.adornUrl(`/sys/ypfl/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
          } else {
            this.$message({ message: data.msg })
          }
        })
      }
    },
    getDrugNameList () {
      this.$http({
        url: this.$http.adornUrl('/sys/common/drugNameList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let drugNameMap = {}
          for (let i = 0; i < data.list.length; i++) {
            drugNameMap = {
              'label': data.list[i].drugName,
              'value': data.list[i].drugId
            }
            this.drugNameList.push(drugNameMap)
          }
          this.rule[1]['options'] = this.drugNameList
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    selectDrugName (val) {
      this.value['drugId'] = val
      for (let i = 0; i < this.drugNameList.length; i++) {
        if (this.drugNameList[i]['value'] === val) {
          this.value['drugName'] = this.drugNameList[i]['label']
          break
        }
      }
    },
    // 取消
    onClose () {
      this.value = {}
      this.initValue = {}
      this.drugNameList = []
      this.functions = ''
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
      this.$http({
        url: this.$http.adornUrl(`/sys/ypcrk/save/${this.functions}`),
        method: 'post',
        data: this.$http.adornData(this.value)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.value = null
          this.initValue = null
          this.dialogVisible = false
          this.drugNameList = []
          this.functions = ''
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
    }
  }
}
</script>
