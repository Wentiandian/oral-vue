<template>
  <div>
<!--    <div v-for="item in this.fileNameList">
      <img style="width: 186px; height: 186px; border:none;cursor: pointer;" :src="getImage(item)" alt=""/>
    </div>-->
    <el-upload
      action="http://localhost:8080/sys/common/upload"
      list-type="picture-card"
      :show-file-list="true"
      :file-list="formData.img"
      :on-preview="handlePictureCardPreview"
      :on-success="handleAvatarSuccess"
      :before-upload="beforeUpload"
      :on-remove="handleRemove"
      :before-remove="beforeRemove">
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt="">
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      fileList: [],
      formData: {
        img: []  // 这里的必须是一个数组，才能正确回显，并且对象中要有name，url，uid
      }
    }
  },
  methods: {
    handleRemove (file, fileList) {
      this.fileList = []
      this.$http({
        url: this.$http.adornUrl(`/sys/common/deleteFile?fileName=${file.name}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message.success(data.msg)
          this.dialogImageUrl = ''
        } else {
          this.$message.success(data.msg)
        }
      }).catch(err => {
        this.$message.error('移除错误' + err)
      })
    },
    beforeRemove () {
      return this.$confirm(`确定移除改图片？移除后无法则恢复`)
    },
    handlePictureCardPreview (file) {
      this.dialogVisible = true
      this.dialogImageUrl = file.url
    },
    handleAvatarSuccess (response, file, fileList) {
      this.dialogImageUrl = file.url
      this.fileList = fileList
    },
    beforeUpload (file) {
      if (file) {
        const size = file.size / 1024 / 1024 < 15
        if (!size) {
          this.$message.error('上传文件大小不能超过 15MB!')
          return false
        }
        return file
      }
    },
    returnFileList () {
      return this.fileList
    },
    onCloseAndSubmit () {
      /* console.log(1414)
      console.log(this.fileList)
      const deleteFiles = []
      for (let i = 0; i < this.fileList.length; i++) {
        deleteFiles.push(this.fileList[i].response.fileName)
      }
      console.log(deleteFiles)
      this.$http({
        url: this.$http.adornUrl(`/sys/common/deleteFiles`),
        method: 'post',
        params: this.$http.adornData(deleteFiles, false)
      }).then(({data}) => {
      }) */
      this.formData.img = []
    },
    fileView (fileList) {
      for (let i = 0; i < fileList.length; i++) {
        this.formData.img.push({name: fileList[i].fileName, url: 'http://localhost:8080/sys/common/download?name=' + fileList[i].fileName})
      }
    }
  }

}
</script>

<style scoped>

</style>
