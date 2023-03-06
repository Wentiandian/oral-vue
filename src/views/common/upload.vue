<template>
  <div>
<!--    <el-upload
      action="http://localhost:8080/sys/common/upload"
      list-type="picture-card"


      :on-success="handleAvatarSuccess"
      :before-upload="beforeUpload"
      :on-preview="handlePictureCardPreview"
      :on-remove="handleRemove">
      <img v-if="imageUrl" :src="imageUrl" class="avatar"></img>
      <i v-else class="el-icon-plus avatar-uploader-icon"></i>
    </el-upload>-->
    <el-upload
      action="http://localhost:8080/sys/common/upload"
      list-type="picture-card"
      :show-file-list="true"
      :on-preview="handlePictureCardPreview"
      :before-upload="beforeUpload"
      :on-remove="handleRemove">
      <i class="el-icon-plus"></i>
    </el-upload>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      fileNameList: []
    }
  },
  methods: {
    handleRemove (file, fileList) {
      console.log(file, fileList)
    },
    handlePictureCardPreview (file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    handleAvatarSuccess (response, file, fileList) {
      // eslint-disable-next-line no-unused-expressions
      this.returnFileList()
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
      console.log(555)
      console.log(this.fileList)
      return this.fileList
    },
    getImage (filename) {
      return `http://localhost:8080/sys/common/download?name=${filename}`
    },
    fileView (fileList) {
      console.log(fileList)
      this.fileNameList = fileList
    }
  }

}
</script>

<style scoped>

</style>
