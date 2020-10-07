<template>
  <el-dialog
    :title="'安排'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <!-- 嘉宾-->
    <el-table
      :data="guests"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="ID">
      </el-table-column>
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="嘉宾姓名">
      </el-table-column>
      <el-table-column
        prop="job"
        header-align="center"
        align="center"
        label="职称">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id,'user')">删除</el-button>
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
    <!--  车辆-->
    <el-table
      :data="cars"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="driverName"
        header-align="center"
        align="center"
        label="司机姓名">
      </el-table-column>
      <el-table-column
        prop="carNumber"
        header-align="center"
        align="center"
        label="车牌号">
      </el-table-column>
      <el-table-column
        prop="carType"
        header-align="center"
        align="center"
        label="车辆类型">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id,'car')">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-autocomplete
      v-model="carState"
      :fetch-suggestions="querySearchCar"
      placeholder="司机姓名"
      @select="handleSelectCar">
    </el-autocomplete>
    <!--  添加按钮-->
    <el-button type="medium" icon="el-icon-plus" @click="addCar"></el-button>
    <!--  分割线-->
    <el-divider></el-divider>
    <!--    志愿者-->
    <el-table
      :data="volunteers"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="志愿者姓名">
      </el-table-column>
      <el-table-column
        prop="telphone"
        header-align="center"
        align="center"
        label="志愿者电话">
      </el-table-column>
      <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="志愿者邮箱">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id,'volunteer')">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-autocomplete
      v-model="volunteerState"
      :fetch-suggestions="querySearchVolunteer"
      placeholder="志愿者姓名"
      @select="handleSelectVolunteer">
    </el-autocomplete>
    <!--  添加按钮-->
    <el-button type="medium" icon="el-icon-plus" @click="addVolunteer"></el-button>
    <!--  分割线-->
    <el-divider></el-divider>
    <!--    房间信息-->
    <el-table
      :data="rooms"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="location"
        header-align="center"
        align="center"
        label="位置">
      </el-table-column>
      <el-table-column
        prop="roomType"
        header-align="center"
        align="center"
        label="房间类型">
      </el-table-column>
      <el-table-column
        prop="roomNumber"
        header-align="center"
        align="center"
        label="房间号">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id,'room')">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-autocomplete
      v-model="roomState"
      :fetch-suggestions="querySearchRoom"
      placeholder="房间号"
      @select="handleSelectRoom">
    </el-autocomplete>
    <!--  添加按钮-->
    <el-button type="medium" icon="el-icon-plus" @click="addRoom"></el-button>
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
      carState: '',
      roomState: '',
      volunteerState: '',
      guestId: '',
      carId: '',
      roomId: '',
      volunteerId: '',
      dataList: [],
      guests: [],
      volunteers: [],
      cars: [],
      rooms: [],
      state: '',
      timeout: null,
      activeId: 0,
      dataListLoading: false
    }
  },
  methods: {
    init (id) {
      this.activeId = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.freshData()
      })
    },
    freshData () {
      this.dataListLoading = true
      if (this.activeId) {
        this.$http({
          url: this.$http.adornUrl(`/app/active/${this.activeId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            let _data = data.data
            console.log(_data)
            this.cars = _data.car
            this.volunteers = _data.volunteer
            this.rooms = _data.room
            this.guests = _data.other
          }
          this.dataListLoading = false
        })
      }
    },
    querySearchGuest (queryString, cb) {
      this.$http({
        url: this.$http.adornUrl(`/app/user`),
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
    querySearchRoom (queryString, cb) {
      this.$http({
        url: this.$http.adornUrl(`/app/room`),
        method: 'get',
        params: this.$http.adornParams({
          'roomNumber': queryString
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          let arr = []
          for (let i = 0; i < data.data.length; i++) {
            arr.push({
              'value': data.data[i].roomNumber + ' ' + data.data[i].location + ' ' + data.data[i].roomType,
              'id': data.data[i].id
            })
          }
          cb(arr)
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    querySearchCar (queryString, cb) {
      this.$http({
        url: this.$http.adornUrl(`/app/car`),
        method: 'get',
        params: this.$http.adornParams({
          'driverName': queryString
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          let arr = []
          for (let i = 0; i < data.data.length; i++) {
            arr.push({
              'value': data.data[i].driverName + ' ' + data.data[i].carNumber + ' ' + data.data[i].carType,
              'id': data.data[i].id
            })
          }
          cb(arr)
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    querySearchVolunteer (queryString, cb) {
      this.$http({
        url: this.$http.adornUrl(`/app/volunteer`),
        method: 'get',
        params: this.$http.adornParams({
          'name': queryString
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          let arr = []
          for (let i = 0; i < data.data.length; i++) {
            arr.push({
              'value': data.data[i].name + ' ' + data.data[i].telphone + ' ' + data.data[i].email,
              'id': data.data[i].id
            })
          }
          cb(arr)
        } else {
          this.$message.error(data.msg)
        }
      })
    },

    deleteHandle (id, type) {
      this.$confirm(`确定进行删除操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl(`/app/active/arrangement`),
          method: 'delete',
          params: this.$http.adornParams({
            'activeId': this.activeId,
            'id': id,
            'e': type
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
    handleSelectCar (item) {
      this.carId = item.id
    },
    handleSelectRoom (item) {
      this.roomId = item.id
    },
    handleSelectVolunteer (item) {
      this.volunteerId = item.id
    },
    // todo
    // 添加安排
    // 自定义插槽
    addGuest () {
      this.arrangeItems(this.guestId, 'user')
    },
    addCar () {
      this.arrangeItems(this.carId, 'car')
    },
    addRoom () {
      this.arrangeItems(this.roomId, 'room')
    },
    addVolunteer () {
      this.arrangeItems(this.volunteerId, 'volunteer')
    },
    arrangeItems (id, type) {
      this.$http({
        url: this.$http.adornUrl(`/app/active/arrangement`),
        method: 'post',
        data: this.$http.adornData({
          'activeId': this.activeId,
          'id': id,
          'e': type
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              // this.getDataList()
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
