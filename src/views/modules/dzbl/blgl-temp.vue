<template>
  <el-dialog
    :title="titleName"
    :close-on-click-modal="false"
    :visible.sync="dialogVisible"
    :before-close="onClose">
    <div>
      <form-create v-model="fApi" :rule="rule" :option="options" :value.sync="value" />
      <el-row :gutter="24">
        <el-col :span="19">
          <div><p>上传图片：</p>
            <Upload ref="Upload"></Upload>
          </div></el-col>
      </el-row>
    </div>
    <div>
      <h3>药物单（处方）</h3>
      <h1>R:</h1>
      <form-create v-model="fApi1" :rule="rule1" :option="options" :value.sync="value1" />
      <el-row :gutter="24">
        <el-col :span="19">
          <div><p></p></div></el-col>
      </el-row>

      <el-button type="primary" icon="el-icon-plus" size="mini" @click="handleAddDetails">添加药品</el-button>
      <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteDetail">删除药品</el-button>
      <el-button type="info" icon="el-icon-delete" size="mini" @click="deleteAllDetails">清空药品</el-button>

      <el-table
        v-loading="dataListLoading"
        :data="drugPreList"
        :row-class-name="rowClassName"
        @selection-change="handleDetailSelectionChange"
        ref="ywd"
      >
        <el-table-column type="selection" width="30" align="center" />
        <el-table-column label="序号" align="center" prop="xh" width="50"></el-table-column>

        <el-table-column label="药品" align="center" prop="drugId">
          <template slot-scope="scope">
            <el-select clearable v-model="drugPreList[scope.row.xh-1].drugId">
              <el-option
                v-for="item in drugNameList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </template>
        </el-table-column>

        <el-table-column label="数量" prop="drugNum">
          <template slot-scope="scope">
            <el-input :style="{width: '100%'}" clearable v-model="drugPreList[scope.row.xh-1].drugNum"></el-input>
          </template>
        </el-table-column>
      </el-table>

    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="dataFormSubmit()">提交</el-button>
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
      dataListLoading: false,
      dialogVisible: false,
      fApi: {},
      fApi1: {},
      isCreate: null,
      rowId: '',
      value: {},
      value1: {},
      initValue: {},
      initValue1: {},
      deptList: [],
      fileList: [],
      drugPreList: [],
      drugNameList: [],
      patientList: [],
      // 选中的从表数据
      checkedDetail: [],
      titleName: '',
      flag: true,
      rule: [
        {
          type: 'input',
          field: 'eleRecordsId',
          title: '病历编号',
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
          field: 'docName',
          title: '医生',
          effect: {
            required: '请填写医生名称'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'nurseName',
          title: '护士',
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'deptId',
          title: '治疗科室',
          options: [],
          effect: {
            required: '请选择治疗科室'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'treatmentNum',
          title: '第几次就诊',
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'isReferral',
          title: '是否复诊',
          effect: {
            required: '请选择是否复诊'
          },
          options: [{label: '是', value: 1}, {label: '否', value: 0}],
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'isGhBooking',
          title: '挂号or预约',
          effect: {
            required: '请选择挂号or预约'
          },
          options: [{label: '挂号', value: 0}, {label: '预约', value: 1}],
          col: {
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'treatmentTime',
          title: '就诊时间',
          effect: {
            required: '请选择就诊时间'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'treatmentDescription',
          title: '医生诊断',
          effect: {
            required: '请填写医生诊断'
          },
          props: {
            type: 'textarea',
            rows: '3'
          }
        },
        {
          type: 'input',
          field: 'treatmentMethod',
          title: '治疗方案',
          effect: {
            required: '请填写治疗方案'
          },
          props: {
            type: 'textarea',
            rows: '3'
          }
        },
        {
          type: 'input',
          field: 'treatmentDuring',
          title: '治疗过程',
          effect: {
            required: '请填写治疗过程'
          },
          props: {
            type: 'textarea',
            rows: '3'
          }
        },
        {
          type: 'input',
          field: 'docOrders',
          title: '医嘱',
          props: {
            type: 'textarea',
            rows: '3'
          }
        }
      ],
      rule1: [
        {
          type: 'input',
          field: 'prescriptionId',
          title: '药物但编号',
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
          type: 'input',
          field: 'hospital',
          title: '医院名称',
          effect: {
            required: '请填写医院名称'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'select',
          field: 'patientId',
          title: '患者名称',
          options: [],
          effect: {
            required: '请填写患者名称'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'select',
          field: 'deptId',
          title: '治疗科室',
          options: [],
          effect: {
            required: '请选择治疗科室'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'datePicker',
          field: 'prescriptionTime',
          title: '开方时间',
          props: {
            disabled: true
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'docName',
          title: '医生',
          effect: {
            required: '请填写医生名称'
          },
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'pharmacist',
          title: '药剂师',
          col: {
            span: 12
          }
        },
        {
          type: 'input',
          field: 'useMethod',
          title: '备注',
          props: {
            type: 'textarea',
            rows: '3'
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
  created () {
    this.getDeptList()
    this.getDrugNameList()
    this.getPatientList()
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
          url: this.$http.adornUrl(`/sys/dzbl/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
            this.value1 = data.prescription
            this.fileList = data.fileList
            this.drugPreList = data.drugPreList
            this.downloadFile()
          } else {
            this.$message({ message: data.msg })
          }
        })
      }
    },
    getDeptList () {
      this.deptList = []
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
          this.rule[4]['options'] = this.deptList
          this.rule1[3]['options'] = this.deptList
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    getDrugNameList () {
      this.drugNameList = []
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
              'value': data.list[i].drugId
            }
            this.drugNameList.push(drugDosageMap)
          }
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    getPatientList () {
      this.patientList = []
      this.$http({
        url: this.$http.adornUrl('/sys/common/patientList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let patientMap = {}
          for (let i = 0; i < data.list.length; i++) {
            patientMap = {
              'label': data.list[i].patientName,
              'value': data.list[i].patientId
            }
            this.patientList.push(patientMap)
          }
          this.rule1[2]['options'] = this.patientList
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    // 取消
    onClose () {
      this.value = {}
      this.initValue = {}
      this.value1 = {}
      this.initValue1 = {}
      this.deptList = []
      this.patientList = []
      this.drugPreList = []
      this.dialogVisible = false
      this.$refs.Upload.onCloseAndSubmit()
      this.fApi.resetFields()
      this.fApi1.resetFields()
      this.$emit('closeHide', false)
      this.$message({ message: '取消编辑' })
    },
    // 重置
    onReset () {
      this.value = this.initValue
      this.value1 = this.initValue1
    },
    // 表单提交
    dataFormSubmit () {
      this.upload()
      this.value['prescriptionEntity'] = this.value1
      this.value['drugPreEntityList'] = this.drugPreList
      this.$http({
        url: this.$http.adornUrl(`/sys/dzbl/${!this.rowId ? 'save' : 'update'}`),
        method: 'post',
        data: this.$http.adornData(this.value)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.value = null
          this.initValue = null
          this.deptList = []
          this.value1 = {}
          this.initValue1 = {}
          this.patientList = []
          this.drugPreList = []
          this.dialogVisible = false
          this.$refs.Upload.onCloseAndSubmit()
          this.fApi.resetFields()
          this.fApi1.resetFields()
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
    upload () {
      let fileList = this.$refs.Upload.returnFileList()
      if (fileList !== undefined) {
        for (let i = 0; i < fileList.length; i++) {
          fileList[i]['status'] = 1
          if (!fileList[i].response) {
            continue
          } else {
            fileList[i]['fileName'] = fileList[i]['response']['fileName']
          }
        }
        this.value['file'] = fileList
      }
    },
    deleteDetail () {
      if (this.checkedDetail.length === 0) {
        this.$alert('请先选择要删除的数据', '提示', {
          confirmButtonText: '确定'
        })
      } else {
        this.drugPreList.splice(this.checkedDetail[0].xh - 1, 1)
      }
    },
    deleteAllDetails () {
      this.drugPreList = undefined
    },
    downloadFile () {
      this.$refs.Upload.fileView(this.fileList)
    },
    rowClassName ({ row, rowIndex }) {
      row.xh = rowIndex + 1
    },
    handleDetailSelectionChange (selection) {
      if (selection.length > 1) {
        this.$refs.ywd.clearSelection()
        this.$refs.ywd.toggleRowSelection(selection.pop())
      } else {
        this.checkedDetail = selection
      }
    },
    handleAddDetails () {
      if (this.drugPreList === undefined) {
        this.drugPreList = []
      }
      let obj = {}
      this.drugPreList.push(obj)
    }
  }
}
</script>
