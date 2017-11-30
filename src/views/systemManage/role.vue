<template>
  <div class="wrap">
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <div style="margin-bottom: 30px">
        <el-form-item label="角色名称">
          <el-input v-model="formInline.userName" placeholder="admin"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" style="padding: 10px 30px" @click="select">查询</el-button>
        </el-form-item>

        <el-button style="padding: 10px 30px" @click="addRow">新增</el-button>
        <el-button style="padding: 10px 30px" @click="refresh">刷新</el-button>
      </div>
      <div class="roleTable">
        <template>
          <el-table
            :data="tableData"
            border
            highlight-current-row
            style="width: 100%">
            <el-table-column
              prop="userId"
              label="序号"
              sortable
              width="180">
            </el-table-column>
            <el-table-column
              prop="userName"
              label="角色名称"
              width="180">
            </el-table-column>
            <el-table-column
              prop="roleBM"
              label="角色编码"
              sortable
              width="360">
            </el-table-column>
            <el-table-column
              prop="createTime"
              label="创建时间"
              sortable
              :sort-method="mySort"
              width="360">
            </el-table-column>
            <el-table-column
              prop="editor"
              label="操作"
              width="">
              <template scope="scope">

                <el-button type="text" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>

                <el-dialog :title="title" :visible.sync="dialogFormVisible" >
                  <div v-if="configId==1">
                    <el-tree
                      :data="data2"
                      show-checkbox
                      indent="16"
                      node-key="id"
                      ref="tree"
                      highlight-current
                      :props="defaultProps">
                    </el-tree>
                  </div>
                  <div v-else-if="editorId==1">
                    <el-form :model="form"
                             style="display: flex;flex-flow: column wrap;justify-content: flex-start;align-items: flex-start">
                      <el-form-item label="序号" :label-width="formLabelWidth" style="text-align: left">
                        <el-input v-model="form.userId" auto-complete="off"></el-input>
                      </el-form-item>
                      <el-form-item label="角色名称" :label-width="formLabelWidth">
                        <el-input v-model="form.userName" auto-complete="off"></el-input>
                      </el-form-item>
                      <el-form-item label="角色编码" :label-width="formLabelWidth">
                        <el-input v-model="form.roleBM" auto-complete="off"></el-input>
                      </el-form-item>
                      <el-form-item label="创建时间" :label-width="formLabelWidth">
                        <el-input v-model="form.createTime" auto-complete="off" disabled></el-input>
                      </el-form-item>
                    </el-form>
                  </div>
                  <div slot="footer" class="dialog-footer">
                    <el-button @click="dialogFormVisible = false">取 消</el-button>
                    <el-button type="primary" @click="sure" v-if="editorId==1">确 定</el-button>
                    <el-button type="primary"  v-else-if="configId==1">确 定</el-button>
                  </div>
                </el-dialog>
                <el-button type="text" size="small" @click="handleConfig(scope.$index, scope.row)">分配</el-button>
                <el-button type="text" size="small" @click="handleDelete(scope.$index, scope.row)"
                           style="margin-left: 0">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </div>
      <div class="block" style="margin:100px auto;display: flex;justify-content: center ">

        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage4"
          :page-sizes="[100, 200, 300, 400]"
          :page-size="100"
          layout="total, sizes, prev, pager, next, jumper"
          :total="400">
        </el-pagination>
      </div>
    </el-form>
  </div>
</template>

<script>
  var index = this.index
  var myDate = new Date()
  var year = myDate.getFullYear()
  var month = myDate.getMonth() + 1
  var day = myDate.getDate()
  var myDay = `${year}/${month}/${day}`
  export default {
    data() {
      return {
        data2: [{
          id: 1,
          label: '一级 1',
          children: [{
            id: 4,
            label: '二级 1-1',
            children: [{
              id: 9,
              label: '三级 1-1-1'
            }, {
              id: 10,
              label: '三级 1-1-2'
            }]
          }]
        }, {
          id: 2,
          label: '一级 2',
          children: [{
            id: 5,
            label: '二级 2-1'
          }, {
            id: 6,
            label: '二级 2-2'
          }]
        }, {
          id: 3,
          label: '一级 3',
          children: [{
            id: 7,
            label: '二级 3-1'
          }, {
            id: 8,
            label: '二级 3-2'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        configId: 0,
        currentPage4: 4,
        editorId: 0,
        index: '',
        title: '',
        dialogFormVisible: false,
        form: {
          userId: '',
          userName: '',
          roleBM: '',
          createTime: '',
        },
        formLabelWidth: '120px',

        formInline: {
          userName: '',
          region: ''
        },
        tableData: [],
        tempArray: []

      }
    },
    mounted() {
      this.tempArray = this.tableData
//      this.displayData()
    },
    methods: {
//      displayData() {
//        var that = this
//        that.service({
//          url: '/roletable',
//          method: 'get'
//        }).then(function (response) {
//          console.log('abc==', response)
//          that.tableData = response.data.tableData
//          console.log(that.tableData)
//        }).catch(function (data) {
//          that.loading = false
//          console.log(data)
//        })
//
//      },
      mySort(a, b) {
        {
          return parseInt(a) - parseInt(b)
        }
      },
      handleSizeChange() {

      },
      handleCurrentChange() {

      },
      select() {
        var temp = []
        if(this.tableData = []) this.tableData = this.tempArray
        for (let i = 0; i < this.tableData.length; i++) {
          if (this.tableData[i].userName.indexOf(this.formInline.userName) != -1) {
            temp.push(this.tableData[i])
            console.log(temp)
          }
        }
        this.tableData = temp
      },
      refresh() {
        this.tableData = this.tempArray
        this.formInline.userName = ''
      },
      handleEdit(index, row) {
        console.log(index, row)
        this.editorId = 1
        this.configId=0
        this.index = index
        this.dialogFormVisible = true
        this.title = '修改用户信息'
        this.form = this.tableData[index]
        this.form.createTime = myDay
      },
      sure(index) {
        this.tableData[index] = this.form
        this.tableData[index].createTime = myDay
        this.dialogFormVisible = false
        this.editorId = 0
        this.form = {}
      },
      addRow() {
        this.dialogFormVisible = true
        this.title = '新增用户信息'
        this.form = {}
        this.form.createTime = myDay
      },
//      submitSure() {
//        this.tableData.push(this.form)
//        this.dialogFormVisible = false
//      },
      handleConfig(index,row){
        this.configId=1
        this.dialogFormVisible = true
        this.title = '管理员分配'

      },
      handleDelete(index, row) {
        console.log(index, row)
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.tableData.splice(index, 1)
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      },
    }
  }
</script>
<style>
  .wrap {
    margin: 50px 0 50px 100px;
    padding-right: 100px;
    box-sizing: border-box;
  }

  .roleTable, .el-table th {
    text-align: center;
  }

  .el-table--border td, .el-table--border th, .el-table td, .el-table th.is-leaf, .el-table {
    border-color: #999;
  }

  .el-table--fit {
    border: 1px solid #999;
  }
</style>
