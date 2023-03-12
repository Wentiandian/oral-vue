<template>
  <el-dialog
    :title="titleName"
    :close-on-click-modal="false"
    :visible.sync="dialogVisible"
    :before-close="onClose">
    <div>
      <form-create v-model="fApi" :rule="rule" :option="options" :value.sync="value" />
      <el-row :gutter="24">
        <el-col :span="19"><div><p /></div></el-col>
      </el-row>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="onClose">确定</el-button>
      <el-button @click="onClose">取消</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      dialogVisible: false,
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
          field: 'orderId',
          title: '订单编号',
          props: {
            disabled: true
          },
          col: {
            span: 12
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
            span: 12
          }
        },
        {
          type: 'input',
          field: 'patientName',
          title: '患者名称',
          props: {
            disabled: true
          },
          col: {
            span: 12
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
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'orderTime',
          title: '生成订单时间',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'createUserId',
          title: '创建人编号',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'createUserName',
          title: '创建人',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'createTime',
          title: '创建时间',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'sum',
          title: '总价',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'select',
          field: 'status',
          title: '是否付款',
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
  methods: {
    open (isCreate, rowId) {
      this.titleName = '订单详细'
      this.rowId = rowId
      this.dialogVisible = true
      this.init()
    },
    init () {
      if (!this.isCreate) {
        this.$http({
          url: this.$http.adornUrl(`/sys/jfgl/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
            this.value['patientName'] = data.patient.patientName
            this.value['createUserName'] = data.user.name
            this.status()
          } else {
            this.$message({ message: data.msg })
          }
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
    },
    status () {
      if (this.value['status'] === '0') {
        this.value['status'] = '未付款'
      } else {
        this.value['status'] = '已付款'
      }
    }
   /* // 重置
    onReset () {
      this.value = this.initValue
    },
    // 表单提交
    dataFormSubmit () {
      this.$http({
        url: this.$http.adornUrl(`/sys/ksgl/${!this.rowId ? 'save' : 'update'}`),
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
    } */
  }
}
</script>
