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
          <div v-for="item in this.fileNameList">
            <img style="width: auto; height: 40px; border:none;cursor: pointer;" :src="getImage(item)" alt=""/>
          </div>
          <div><p>上传图片：</p>
            <Upload ref="Upload"></Upload>
          </div></el-col>
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
      rowId: '',
      value: {},
      initValue: {},
      deptList: [],
      fileNameList: [],
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
      options: {
        submitBtn: false,
        resetBtn: false
      }
    }
  },
  watch: {
    isCreate () {
      this.getDeptList()
      if (this.isCreate) this.titleName = '新增'
      else this.titleName = '编辑'
    }
  },
  created () {
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
            this.fileList = data.fileList
            this.downloadFile()
          } else {
            this.$message({ message: data.msg })
          }
        })
      }
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
          this.rule[4]['options'] = this.deptList
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    // 取消
    onClose () {
      this.value = {}
      this.initValue = {}
      this.deptList = []
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
      this.upload()
      this.$http({
        url: this.$http.adornUrl(`/sys/dzbl/${!this.rowId ? 'save' : 'update'}`),
        method: 'post',
        data: this.$http.adornData(this.value)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.value = null
          this.initValue = null
          this.deptList = []
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
    upload () {
      let fileList = this.$refs.Upload.returnFileList()
      console.log(222)
      console.log(fileList)
      for (let i = 0; i < fileList.length; i++) {
        fileList[i]['fileName'] = fileList[i]['response']['fileName']
      }
      this.value['file'] = fileList
    },
    downloadFile () {
      for (let i = 0; i < this.fileList.length; i++) {
        this.fileNameList.push(this.fileList[i]['fileName'])
      }
      console.log(11111111)
      console.log(this.fileNameList)
      this.$refs.Upload.fileView(this.fileNameList)
    },
    getImage (filename) {
      return `http://localhost:8080/sys/common/download?name=${filename}`
    }
  }
}
</script>
