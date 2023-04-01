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
import {multiple} from 'webpack-merge'

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
      roleList: [],
      flag: true,
      rule: [
        {
          type: 'input',
          field: 'userId',
          title: '医护工作者编号',
          props: {
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'input',
          field: 'username',
          title: '用户名',
          props: {
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'input',
          field: 'name',
          title: '医护工作者名称',
          props: {
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'select',
          field: 'roleIdList',
          title: '角色',
          options: [],
          effect: {
            required: true
          },
          props: {
            multiple,
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'input',
          field: 'mobile',
          title: '手机号',
          props: {
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'input',
          field: 'email',
          title: '邮箱',
          props: {
            disabled: true
          },
          col: {
            span: 24
          }
        },
        {
          type: 'select',
          field: 'sex',
          title: '性别',
          effect: {
            required: true
          },
          options: [{label: '男', value: '1'}, {label: '女', value: '0'}],
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
          title: '状态',
          effect: {
            required: true
          },
          options: [{label: '启用', value: 1}, {label: '禁用', value: 0}],
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
  created () {
    this.getRoleList()
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
          url: this.$http.adornUrl(`/sys/yhgl/info/${this.rowId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.initValue = data.info
            this.value = data.info
            let roleList = []
            for (let i = 0; i < data.roleList.length; i++) {
              roleList.push(data.roleList[i].roleId)
            }
            this.value['roleIdList'] = roleList
          } else {
            this.$message({message: data.msg})
          }
        })
      }
    },
    getRoleList () {
      this.roleList = []
      this.$http({
        url: this.$http.adornUrl('/sys/common/roleList'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          let roleMap = {}
          for (let i = 0; i < data.list.length; i++) {
            roleMap = {
              'label': data.list[i].roleName,
              'value': data.list[i].roleId
            }
            this.roleList.push(roleMap)
          }
          this.rule[3]['options'] = this.roleList
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
        url: this.$http.adornUrl(`/sys/yhgl/update`),
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
