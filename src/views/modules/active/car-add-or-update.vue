<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="司机姓名" prop="driverName">
      <el-input v-model="dataForm.driverName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="车牌号" prop="carNumber">
      <el-input v-model="dataForm.carNumber" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="车辆类型" prop="carType">
      <el-input v-model="dataForm.carType" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="司机电话" prop="driverTelphone">
      <el-input v-model="dataForm.driverTelphone" placeholder=""></el-input>
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
        dataForm: {
          id: 0,
          driverName: '',
          carNumber: '',
          carType: '',
          driverTelphone: ''
        },
        dataRule: {
          driverName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          carNumber: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          carType: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          driverTelphone: [
            { required: true, message: '不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/app/car/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.driverName = data.car.driverName
                this.dataForm.carNumber = data.car.carNumber
                this.dataForm.carType = data.car.carType
                this.dataForm.driverTelphone = data.car.driverTelphone
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/app/car/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'driverName': this.dataForm.driverName,
                'carNumber': this.dataForm.carNumber,
                'carType': this.dataForm.carType,
                'driverTelphone': this.dataForm.driverTelphone
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
      }
    }
  }
</script>
