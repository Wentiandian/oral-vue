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
    <h3>患者病历信息</h3>
    <el-table
      :data="tableData"
      style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="患者名称">
              <span>{{ props.row.patientName }}</span>
            </el-form-item>
            <el-form-item label="医生">
              <span>{{ props.row.docName }}</span>
            </el-form-item>
            <el-form-item label="护士">
              <span>{{ props.row.nurseName }}</span>
            </el-form-item>
            <el-form-item label="科室编号">
              <span>{{ props.row.deptId }}</span>
            </el-form-item>
            <el-form-item label="治疗科室">
              <span>{{ props.row.deptId }}</span>
            </el-form-item>
            <el-form-item label="诊断描述">
              <span>{{ props.row.treatmentDescription }}</span>
            </el-form-item>
            <el-form-item label="治疗方案">
              <span>{{ props.row.treatmentMethod }}</span>
            </el-form-item>
            <el-form-item label="治疗过程">
              <span>{{ props.row.treatmentDuring }}</span>
            </el-form-item>
            <el-form-item label="医嘱">
              <span>{{ props.row.docOrders }}</span>
            </el-form-item>
            <el-form-item label="药物单编号">
              <span>{{ props.row.prescriptionId }}</span>
            </el-form-item>
            <el-form-item label="医院名称">
              <span>{{ props.row.prescriptionEntity.hospital }}</span>
            </el-form-item>
            <el-form-item label="开处方时间">
              <span>{{ props.row.prescriptionEntity.prescriptionTime }}</span>
            </el-form-item>
            <el-form-item label="使用方法">
              <span>{{ props.row.prescriptionEntity.useMethod }}</span>
            </el-form-item>
            <el-form-item label="药剂师">
              <span>{{ props.row.prescriptionEntity.pharmacist }}</span>
            </el-form-item>
            <el-form-item>
              <h3>开药情况</h3>
            </el-form-item>
            <el-form-item>
              <span></span>
            </el-form-item>
            <el-form-item label="药品"/>
            <el-form-item label="数量"/>
            <el-form-item>
              <div v-for="item in props.row.drugPreEntityList">
                <span>{{ item.drugName }}</span>
              </div>
            </el-form-item>
            <el-form-item>
              <div v-for="item in props.row.drugPreEntityList">
                <span>{{ item.drugNum }}</span>
              </div>
            </el-form-item>

          </el-form>
        </template>
      </el-table-column>
      <el-table-column label="病历编号" prop="eleRecordsId"></el-table-column>
      <el-table-column label="第几次就诊" prop="treatmentNum"></el-table-column>
      <el-table-column label="就诊时间" prop="treatmentTime"></el-table-column>
    </el-table>
  </el-dialog>

</template>

<script>
export default {
  data () {
    return {
      dialogVisible: false,
      fApi: {},
      rowId: '',
      value: {},
      initValue: {},
      isCreate: null,
      titleName: '',
      flag: true,
      tableData: [],
      rule: [
        {
          type: 'input',
          field: 'patientId',
          title: '患者编号',
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
          field: 'patientName',
          title: '患者名称',
          effect: {
            required: '请填写患者名称'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'mobile',
          title: '手机号',
          effect: {
            required: '请填写手机号'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'email',
          title: '邮箱',
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'sex',
          title: '性别',
          effect: {
            required: '请选择性别'
          },
          options: [{label: '♂男', value: 1}, {label: '♀女', value: 0}],
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'age',
          title: '年龄',
          effect: {
            required: '请填写年龄'
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
          options: [{label: '启用', value: 1}, {label: '禁用', value: 0}],
          col: {
            span: 8
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
      this.isCreate = isCreate
      this.titleName = '患者信息'
      this.dialogVisible = true
      this.rowId = rowId
      this.init()
    },
    init () {
      this.$http({
        url: this.$http.adornUrl(`/sys/hzgl/info/${this.rowId}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.initValue = data.info
          this.value = data.info
          this.tableData = data.list
        } else {
          this.$message({ message: data.msg })
        }
      })
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
        url: this.$http.adornUrl(`/sys/hzgl/update`),
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
<style>
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
</style>
