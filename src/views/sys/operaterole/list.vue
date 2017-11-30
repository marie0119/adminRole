<template>
  <div class="wrap">
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <div style="margin-bottom: 30px;overflow: hidden">
        <div  style="float: left">
          <el-form-item label="角色名称">
            <el-input v-model="formInline.userName" placeholder="admin"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" style="padding: 10px 30px" @click="select">查询</el-button>
          </el-form-item>
        </div>
        <div style="float: right">
          <el-button style="padding: 10px 30px" @click="addRow">新增</el-button>
          <el-button style="padding: 10px 30px" @click="refresh">刷新</el-button>
        </div>

      </div>
      <div class="roleTable">
        <template>
          <el-table
            :data="tableData"
            border
            highlight-current-row
            v-loading="loading"
            style="width: 100%">
            <el-table-column align="center" label="序号" width="100">
              <template scope="scope">
                {{scope.$index+1}}
              </template>
            </el-table-column>
            <el-table-column
              prop="roleName"
              label="角色名称"
              width="200">
            </el-table-column>
            <el-table-column
              prop="roleCode"
              label="角色编码"
              width="200">
            </el-table-column>
            <el-table-column
              prop="createDate"
              label="创建时间"
              width="250">
            </el-table-column>
            <el-table-column
              prop="editor"
              label="操作"
              width="auto">
              <template scope="scope">
                <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                <el-button  size="small" @click="handleConfig(scope.$index, scope.row)" style="margin-left: 0">分配</el-button>
                <el-button  size="small" @click="roleConfig(scope.$index, scope.row)" style="margin-left: 0">人员分配</el-button>
                <el-button type="danger" size="small" @click="handleDelete(scope.$index, scope.row)" style="margin-left: 0">删除 </el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-dialog :title="title" :visible.sync="dialogFormVisible" :before-close="handleClose" :size="dialogSize" v-if="dialogId==1">
            <div v-if="configId==1">
              <el-tree
                :data="data3"
                show-checkbox

                node-key="id"
                ref="tree"
                highlight-current
                :default-expanded-keys="expandList"
                :default-checked-keys="checkedList"
                :props="defaultProps"
                style="max-height: 500px;overflow-y: scroll"
              >
              </el-tree>
            </div>
            <div v-else-if="roleConfigId==1">
              <template>
                <el-transfer  :filter-method="filterMethod" :titles="['未选择', '已选择']" v-model="value1" filterable :data="data"></el-transfer>
              </template>

            </div>
            <div v-else>
              <el-form :model="form" :rules="roleRules" ref="form"
                       style="display: flex;flex-flow: column wrap;justify-content: flex-start;align-items: center;">
                <el-form-item label="角色名称" prop="roleName" :label-width="formLabelWidth" style="display: flex">
                  <el-input v-model="form.roleName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="角色编码" prop="roleCode" :label-width="formLabelWidth"  style="display: flex">
                  <el-input v-model="form.roleCode" auto-complete="off"></el-input>
                </el-form-item>
              </el-form>
            </div>
            <div slot="footer" class="dialog-footer">
              <div class="configSave" v-if="configId==1"><el-button type="primary"  @click="saveConfigs">保存</el-button></div>
              <div v-else>
              <el-button @click="cancelEvent">取 消</el-button>
              <el-button type="primary" @click="editorSave('form')" v-if="editorId==1">确 定</el-button>
                <el-button type="primary" @click="saveOption"  v-else-if="roleConfigId==1">保存</el-button>
              <el-button type="primary" @click="addSave('form')" v-else>确 定</el-button>
              </div>
            </div>
          </el-dialog>
        </template>
      </div>
      <div class="block" style="margin:100px 0;">

        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>
    </el-form>
  </div>
</template>

<script>
  export default {
//    name:'角色管理',
    data() {
      var checkRole = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('角色名称不能为空'));
        }
        setTimeout(() => {
          if (value.length>20) {
            callback(new Error('角色名称长度不超过20'));
          }  else {
            callback();
          }
        }, 1000);
      };
      var checkRoleCode = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('角色编码不能为空'));
        }
        setTimeout(() => {
          if (value.length>20) {
            callback(new Error('角色编码长度不超过20'));
          }  else {
            callback();
          }
        }, 1000);
      };


      return {
        dialogId:0,
        data: [],
        value1: [],
        dialogSize:'tiny',
        filterMethod(query, item) {
          return item.label.indexOf(query) > -1;
        },
        roleConfigId:0,
        index:'',
        id: '',
        loading:true,
        data3: [],
        checkedList: [],
        expandList: [],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        total:null,
        configId: 0,
        currentPage: 1,
        pageSize:10,
        editorId: 0,
        title: '',
        dialogFormVisible: false,
        form: {
          roleCode: '',
          roleName: ''
        },
        formLabelWidth: 'auto',
        formInline: {
          userName: '',
          region: ''
        },
        tableData: [],
        roleRules: {
          roleName: [
            { validator: checkRole, trigger: 'blur' }
          ],
          roleCode: [
            { validator: checkRoleCode, trigger: 'blur' }
          ],
        }
      }
    },
    mounted() {
      this.displayData()
    },
    methods: {
      roleConfig(index,row){ //人员分配
        this.dialogId=1
        this.roleConfigId=1
        this.dialogFormVisible=true
        this.title='管理员分配人员'
        this.dialogSize='small'
        let that=this
        that.id=row.id
        that.service({
          url: '/sys/operaterole/allotOperateUser',
          method: 'post',
          data: ({
            id: row.id
          })
        }).then(function (response) {
          console.log('穿梭:', response)
          that.data=response.data.allUser
          that.value1=response.data.selectedUser
        }).catch(function (data) {
          console.log(data)
        })
      },
      saveOption(){ //人员分配保存
        let that=this
        that.service({
          url: '/sys/operaterole/allotOperateUserSave',
          method: 'post',
          data: ({
            id: that.id,
            checkedOperateUser:that.value1.join()
          })
        }).then(function (response) {
          console.log('value1:',that.value1.join())
          console.log('id:',that.id)
          console.log('穿梭保存:', response)
          that.dialogId=0
          that.dialogFormVisible=false
          that.roleConfigId=0
          that.$message({
            type: 'success',
            message: response.message
          });
          that.dialogSize='tiny'
        }).catch(function (data) {
          console.log(data)
        })
      },
      handleClose(){
        this.dialogFormVisible = false
        this.dialogId=0
        this.editorId = 0
        this.configId = 0
        this.roleConfigId=0
        this.dialogSize='tiny'
      },
      cancelEvent(){ //-------------取消
        this.dialogFormVisible = false
        this.dialogId=0
        this.editorId = 0
        this.configId = 0
        this.roleConfigId=0
        this.dialogSize='tiny'
      },
      handleConfig(index, row) { //---------------------角色分配
        this.id = row.id
        this.dialogId=1
        this.configId = 1
        this.dialogFormVisible = true
        this.title = '管理员分配'
        var that = this
        that.service({
          url: '/sys/operaterole/allotOperateMenu',
          method: 'post',
          data: ({
            id: row.id
          })
        }).then(function (response) {
          console.log('分配:', response)
          that.data3 = response.data.jsonData.rootList
          that.checkedList = response.data.jsonData.checkedList
          that.expandList = response.data.jsonData.expandList
        }).catch(function (data) {
          that.loading = false
          console.log(data)
        })
      },
      saveConfigs() {       //---------------------角色分配 保存
        var that = this
        var tempData = this.getCheckedNodes();
        console.log(tempData)
        tempData.id = that.id;
        that.service({
          url: '/sys/operaterole/allotOperateMenuSave',
          method: 'post',
          data: (tempData)
        }).then(function (response) {
          that.$message({
            message:response.message,
            type:'success'
          })
          that.dialogId=0
          that.dialogFormVisible=false;
          that.configId=0
        }).catch(function (data) {
          that.loading = false
          console.log(data)
        })
      },
      getCheckedNodes() { //---------------------角色分配 获取节点方法
        var that = this
        console.log('节点数组:', that.$refs.tree.getCheckedNodes());
        var temp = that.$refs.tree.getCheckedNodes()
        var m = []
        var txt = []
        for (let i = 0; i < temp.length; i++) {
          if (temp[i].is_menu)
            m.push(temp[i].id)
          else {
            txt.push(temp[i].id)
          }
        }
        return {checkedOperateMenu: m.join(), checkedOperateMenuBtn: txt.join()}
      },
      displayData() { //---------------------角色 表格显示
        var that = this
        that.service({
          url: '/sys/operaterole/listData',
          method: 'post',
          data:({
            pageSize:that.pageSize,
            pageId:that.currentPage
          })
        }).then(function (response) {
          that.tableData = response.data.rows
          that.total=response.data.rowCount
          that.loading=false
        }).catch(function (data) {
        })
      },
      handleSizeChange(val) {
        this.pageSize=val
        this.displayData()
      },
      handleCurrentChange(val) {
        this.currentPage=val
        this.displayData()
      },
      select() { //---------------------查询
        var that = this
        that.service({
          url: '/sys/operaterole/listData',
          method: 'post',
          data:({
            pageSize:that.pageSize,
            pageId:that.currentPage,
            roleNameSearch:that.formInline.userName
          })
        }).then(function (response) {
          console.log('select:',response)
          that.tableData = response.data.rows
          that.formInline.userName=''
        }).catch(function (data) {

        })
      },
      refresh() { //---------------------刷新
      this.displayData()
      },
      handleEdit(index, row) { //---------------------编辑
        this.editorId = 1
        this.dialogId=1
        this.configId = 0
        this.dialogFormVisible = true
        this.title = '修改用户信息'
        this.id=row.id
        this.index=index
        this.form = this.tableData[index]
        var that = this
        console.log('id:',row.id)
        that.service({
          url: '/sys/operaterole/update',
          method: 'post',
          data:({
            id:row.id
          })
        }).then(function (response) {
          console.log('editor:',response)
        }).catch(function (data) {

        })
      },



      editorSave(formName) { //---------------------编辑 保存
        var that = this
        let id=that.id
        let index=that.index
        this.$refs[formName].validate((valid) => {
          if (valid) {
            that.service({
              url: '/sys/operaterole/updateSave',
              method: 'post',
              data:({
                id:id,
                roleName:that.form.roleName,
                roleCode:that.form.roleCode
              })
            }).then(function (response) {
              that.dialogFormVisible = false
              that.editorId = 0
              that.dialogId=0
              that.$message({
                message:response.message,
                type:'success'
              })
              that.displayData()
            }).catch(function (data) {

            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });


      },
      addRow(formName) { //---------------------新增
        this.form={}
        this.dialogId=1
        this.dialogFormVisible = true
        this.title = '新增用户信息'
        let that=this
        that.editorId = 0
        that.service({
          url: '/sys/operaterole/add',
          method: 'post',
          data:({
          })
        }).then(function (response) {

        }).catch(function (data) {

        })
      },

      addSave(formName) { //---------------------新增 保存
        let that=this
        this.$refs[formName].validate((valid) => {
          if (valid) {
            that.service({
              url: '/sys/operaterole/addSave',
              method: 'post',
              data:({
                roleName:that.form.roleName,
                roleCode:that.form.roleCode
              })
            }).then(function (response) {
              console.log('save:',response)
              that.dialogId=0
              that.dialogFormVisible = false
              that.$message({
                message:response.message,
                type:'success'
              })
              that.displayData()
            }).catch(function (data) {
              this.dialogFormVisible =false
            })
          } else {
            console.log('error submit!!');
            return false;
          }
        });

      },
      handleDelete(index, row) { //---------------------删除
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let that=this
          that.service({
            url: '/sys/operaterole/delete',
            method: 'post',
            data:({
              id:row.id
            })
          }).then(function (response) {
            that.$message({
              type: 'success',
              message: '删除成功!'
            })
            that.displayData()
          }).catch(function (data) {

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
  .el-transfer{
    display: flex;
    justify-content:center;
    align-items: center;
  }
  .el-form-item__content{
    margin-left: 0!important;
  }
  .wrap {
    margin: 50px 0 50px 20px;
    padding-right: 100px;
    box-sizing: border-box;
  }

  .roleTable, .el-table th {
    text-align: center;
  }
  .el-dialog__body{
    text-align: left;
  }

  .el-dialog--small {
    height: 500px;
    overflow-y: auto;
    /*width: 20%;*/
  }

  .el-dialog__header {
    text-align: center;
  }

  .el-tree {
    border: none;
  }
.configSave{
  position: absolute;
  right: 50px;
  top: 10px;
}
  .configSave button{
    padding: 10px 15px!important;
  }
  .el-dialog__footer{
    display: flex;
    justify-content: center;
  }
  .dialog-footer button{
    padding: 10px 30px;
  }

</style>
