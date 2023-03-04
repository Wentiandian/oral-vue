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
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      <el-button type="warning" plain aligen="center" @click="onReset">重置</el-button>
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
          field: 'deptId',
          title: '科室编号',
          effect: {
            required: true
          },
          props: {
            disabled: true
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'deptName',
          title: '科室名称',
          effect: {
            required: '请填写科室名称'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'director',
          title: '科室主任',
          effect: {
            required: '请填写科室主任'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'docNum',
          title: '医护人数',
          effect: {
            required: '请填写医护人数'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'deptRoom',
          title: '科室房间',
          effect: {
            required: '请填写科室房间'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'residualNum',
          title: '剩余预约人数',
          effect: {
            required: '请填写剩余预约人数'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'status',
          title: '状态',
          effect: {
            required: '请选择状态'
          },
          options: [{label: '启用', value: 1}, {label: '禁用', value: 0}
          ],
          col: {
            span: 8
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
            span: 8
          }
        },
        {
          type: 'input',
          field: 'description',
          title: '科室介绍',
          props: {
            type: 'textarea',
            rows: '7'
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
      else this.titleName = '编辑'
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
        this.$http({
          url: this.$http.adornUrl(`/sys/ksgl/info/${this.rowId}`),
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
    }
  }
}
</script>
