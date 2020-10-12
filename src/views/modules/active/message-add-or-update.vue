<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="用户名" prop="toUser">
        <el-autocomplete
          v-model="userState"
          :fetch-suggestions="querySearchUser"
          placeholder="用户"
          @select="handleSelectGuest">
        </el-autocomplete>
      </el-form-item>

      <el-form-item label="内容" prop="content">
        <el-input v-model="dataForm.content" placeholder=""></el-input>
      </el-form-item>

    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>

  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      visible: false,
      userState: '所有人',
      dataForm: {
        id: 0,
        fromUser: '',
        toUser: '',
        content: '',
        createTime: '',
        read: ''
      },
      dataRule: {
        content: [
          {required: true, message: '不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    init (id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/app/message/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.from = data.message.from
              this.dataForm.to = data.message.to
              this.dataForm.content = data.message.content
              this.dataForm.createTime = data.message.createTime
              this.dataForm.read = data.message.read
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit () {
      // 设置取反
      if (this.dataForm.toUser == null || this.dataForm.toUser === '') {
        this.dataForm.toUser = -1
      }
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/app/message/${!this.dataForm.id ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'fromUser': 0,
              'toUser': this.dataForm.toUser,
              'content': this.dataForm.content
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      })
    },
    querySearchUser (queryString, cb) {
      if (queryString === '所有人') {
        this.dataForm.toUser = -1
        this.userState = '所有人'
      } else {
        // 查询用户
        this.$http({
          url: this.$http.adornUrl(`/app/user/`),
          method: 'get',
          params: this.$http.adornParams({
            'username': queryString
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            let arr = []
            for (let i = 0; i < data.data.length; i++) {
              arr.push({
                'value': data.data[i].username + ' ' + data.data[i].job + ' ' + data.data[i].mobile,
                'id': data.data[i].userId
              })
            }
            cb(arr)
          } else {
            this.$message.error(data.msg)
          }
        })
      }
    },
    handleSelectGuest (item) {
      this.dataForm.toUser = item.id
    }
  }
}
</script>
