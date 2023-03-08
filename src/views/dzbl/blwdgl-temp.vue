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
          <div><p>上传文件：</p>
            <el-upload
              class="upload-demo"
              drag
              action="http://localhost:8080/sys/common/upload"
              multiple
              :before-upload="beforeUpload"
              :on-preview="handlePictureCardPreview"
              :on-success="handleAvatarSuccess">
              <i class="el-icon-upload"></i>
              <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
              <div class="el-upload__tip" slot="tip">只能上传pdf/txt文件，且不超过1Mb</div>
            </el-upload>
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
      relationId: '',
      value: {},
      initValue: {},
      titleName: '',
      flag: true,
      rule: [
        {
          type: 'input',
          field: 'fileId',
          title: '病历编号',
          effect: {
            required: '请填写病历编号'
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
          field: 'relationId',
          title: '病历编号',
          effect: {
            required: '请填写病历编号'
          },
          col: {
            span: 8
          }
        },
        {
          type: 'datePicker',
          field: 'fileTime',
          title: '上传时间',
          props: {
            disabled: true
          },
          col: {
            span: 10
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
  },
  methods: {
    open (isCreate, rowId, relationId) {
      this.isCreate = isCreate
      this.rowId = rowId
      this.relationId = relationId
      this.dialogVisible = true
      this.init()
    },
    init () {
      if (!this.isCreate) {
        this.$http({
          url: this.$http.adornUrl(`/sys/blwdgl/info/${this.rowId}/${this.relationId}`),
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
      console.log(1111)
      console.log(this.value)
      this.$http({
        url: this.$http.adornUrl(`/sys/blwdgl/${!this.rowId ? 'save' : 'update'}`),
        method: 'post',
        data: this.$http.adornParams(this.value, false)
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
    handleAvatarSuccess (response, file, fileList) {
      // eslint-disable-next-line no-unused-vars
      let list = []
      for (let i = 0; i < fileList.length; i++) {
        let fileValue = {
          'fileName': fileList[i].response.fileName,
          'size': fileList[i].size,
          'status': 0
        }
        list.push(fileValue)
      }
      this.value['fileList'] = list
    },
    beforeUpload (file) {
      if (file) {
        const size = file.size / 1024 / 1024 < 1
        if (!size) {
          this.$message.error('上传文件大小不能超过 1MB!')
          return false
        }
        return file
      }
    },
    handlePictureCardPreview (file) {
      this.dialogVisible = true
      this.dialogImageUrl = file.url
    },
    downloadFile () {
      this.$refs.Upload.fileView(this.fileList)
    }
  }
}
</script>
