<template>
  <el-dialog
    :title="'安排'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <!-- 嘉宾-->
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="job"
        header-align="center"
        align="center"
        label="职位">
      </el-table-column>
      <el-table-column
        prop="job"
        header-align="center"
        align="center"
        label="电话">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.userId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-autocomplete
      v-model="guestState"
      :fetch-suggestions="querySearchGuest"
      placeholder="嘉宾姓名"
      @select="handleSelectGuest">
    </el-autocomplete>
    <!--  添加按钮-->
    <el-button type="medium" icon="el-icon-plus" @click="addGuest"></el-button>
    <!--  分割线-->
    <el-divider></el-divider>
  </el-dialog>

</template>

<script>
export default {
  data () {
    return {
      visible: false,
      restaurants: [],
      guestState: '',
      guestId: '',
      dataList: [],
      guests: [],
      state: '',
      timeout: null,
      uid: 0,
      dataListLoading: false
    }
  },
  methods: {
    init (id) {
      this.uid = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.freshData()
      })
    },
    freshData () {
      // 清空数据
      this.guestState = ''
      this.dataListLoading = true
      if (this.uid) {
        this.$http({
          url: this.$http.adornUrl(`/app/user/arrangement/${this.uid}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.data
          }
          this.dataListLoading = false
        })
      }
    },
    querySearchGuest (queryString, cb) {
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
    },
    deleteHandle (id) {
      this.$confirm(`确定进行删除操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        console.log(id)
        this.$http({
          url: this.$http.adornUrl(`/app/user/arrangement`),
          method: 'delete',
          params: this.$http.adornParams({
            'uid': id,
            'vid': this.uid
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.freshData()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    handleSelect (item) {
      console.log(item)
    },
    handleSelectGuest (item) {
      this.guestId = item.id
    },
    // todo
    // 添加安排
    // 自定义插槽
    addGuest () {
      this.arrangeItems(this.guestId)
    },

    arrangeItems (id) {
      this.$http({
        url: this.$http.adornUrl(`/app/user/arrangement`),
        method: 'post',
        params: this.$http.adornParams({
          'uid': id,
          'vid': this.uid
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.freshData()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    }
  }
}
</script>
