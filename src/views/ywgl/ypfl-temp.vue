<template>
  <el-dialog
    :title="titleName"
    :close-on-click-modal="false"
    :visible.sync="dialogVisible"
    :before-close="onClose">
    <div>
      <form-create v-model="fApi" :rule="rule" :option="options" :value.sync="value" />
      <el-row :gutter="24">
        <el-col :span="19"><div><p>上传药品图片(一张):</p>
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
      drugDosageList: [],
      titleName: '',
      flag: true,
      rule: [
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
            span: 8
          }
        },
        {
          type: 'input',
          field: 'drugName',
          title: '药品名称',
          effect: {
            required: '请填写药品名称'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'select',
          field: 'drugDosageForm',
          title: '药品剂型',
          options: [],
          effect: {
            required: '请选择药品剂型'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'drugSize',
          title: '药品规格',
          effect: {
            required: '请填写药品规格'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'price',
          title: '药品单价',
          effect: {
            required: '请填写药品单价'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'inventory',
          title: '库存量',
          effect: {
            required: '请填写库存量'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'usageDosage',
          title: '用法用量',
          effect: {
            required: '请填写用法用量'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'address',
          title: '存放地址',
          effect: {
            required: '请填写存放地址'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'lifeTime',
          title: '保质期',
          effect: {
            required: '请填写保质期'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'input',
          field: 'description',
          title: '药品描述',
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
      this.getDrugDosageList()
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
          url: this.$http.adornUrl(`/sys/ypfl/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
            this.downloadFile()
          } else {
            this.$message({ message: data.msg })
          }
        })
      }
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
          this.rule[2]['options'] = this.drugDosageList
        } else {
          this.$message({ message: data.msg })
        }
      })
    },
    // 取消
    onClose () {
      this.value = {}
      this.initValue = {}
      this.drugDosageList = []
      this.dialogVisible = false
      this.$refs.Upload.onCloseAndSubmit()
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
        url: this.$http.adornUrl(`/sys/ypfl/${!this.rowId ? 'save' : 'update'}`),
        method: 'post',
        data: this.$http.adornData(this.value)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.value = null
          this.initValue = null
          this.dialogVisible = false
          this.drugDosageList = []
          this.$refs.Upload.onCloseAndSubmit()
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
      console.log(fileList)
      if (fileList.length > 1) {
        this.$message({message: '只能上传一张图片', type: 'danger'})
      } else if (fileList.length > 0) {
        this.value['imageName'] = fileList[0]['response']['fileName']
      }
      console.log(this.value)
    },
    downloadFile () {
      if (this.value['imageName'] !== null) {
        this.$refs.Upload.fileView([{'fileName': this.value['imageName']}])
      }
    }
  }
}
</script>
