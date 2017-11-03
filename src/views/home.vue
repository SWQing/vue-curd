<template>
  <div>
    <el-container>
      <el-header>
        <h1>Vue + Node CURD增删改查项目</h1>
      </el-header>
      <el-main>
        <el-row>
          <el-col :span="22">
            <div class="fr margin40">
              <el-button size="small" type="primary" icon="el-icon-plus" @click="addDialog = true">添加</el-button>
              <el-button size="small" type="danger" icon="el-icon-delete" @click="deletesButton">删除</el-button>
            </div>
          </el-col>
          <div class="clearfix"></div>
        </el-row>
        <el-row>
          <el-col :span="22" :offset="1">
            <el-table :data="userList" tooltip-effect="dark" style="width: 100%" @selection-change="selectButton">
              <el-table-column type="selection" width="55"></el-table-column>
              <el-table-column prop="username" width="100" label="用户名"></el-table-column>
              <el-table-column sortable prop="name" width="80" label="姓名"></el-table-column>
              <el-table-column prop="phone" label="手机"></el-table-column>
              <el-table-column prop="email" label="邮箱"></el-table-column>
              <el-table-column sortable prop="create_time" label="注册日期"></el-table-column>
              <el-table-column prop="is_active" label="状态" width="75">
                <template slot-scope="scope">
                  <el-tag :type="scope.row.is_active == true ? 'success':'danger'">{{scope.row.is_active | formatter}}</el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="250">
                <template slot-scope="scope">
                  <el-button type="success" size="small"  @click="setUser(scope.row)">编辑</el-button>
                  <el-button type="danger" size="small" @click="deleteUser(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <div class="block">
              <el-pagination layout="prev, pager, next" :total="total" :page-size="5" @current-change="pageChange"></el-pagination>
            </div>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
    <el-dialog title="添加新用户" :visible.sync="addDialog" @close="resetForm('addForm')">
      <el-form :model="addForm" :rules="addRules" ref="addForm" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input type="text" v-model="addForm.username" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input type="text" v-model="addForm.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="addForm.password" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="repeat_password">
          <el-input type="password" v-model="addForm.repeat_password" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="phone">
          <el-input type="text" v-model.number="addForm.phone" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input type="text" v-model="addForm.email" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="是否启用" prop="is_active">
          <el-switch v-model="addForm.is_active" auto-complete="off"></el-switch>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="small" @click="submitForm('addForm')">提交</el-button>
          <el-button size="small" @click="resetForm('addForm')">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <el-dialog title="修改用户" :visible.sync="editDialog" @close="resetForm('editForm')">
      <el-form :model="editForm" :rules="addRules" ref="editForm" label-width="100px">
        <el-form-item label="姓名" prop="name">
          <el-input type="text" v-model="editForm.name"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="phone">
          <el-input type="text" v-model.number="editForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input type="text" v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="是否启用" prop="is_active">
          <el-switch v-model="editForm.is_active"></el-switch>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="small" @click="updateUser">修改</el-button>
          <el-button size="small" @click="resetForm('editForm')">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
  import axios from 'axios';
  export default {
    name: 'Home',
    mounted: function () {
      this.getUsers()
    },
    //组件渲染完毕后调用getUsers, 来获取所有的用户列表
    data() {
      //自定义的验证规则
      var checkPass = (rule, value, callback) => {
        if(value === '') {
            callback(new Error('确认密码不能为空'));
        }
        else if(value !== this.addForm.password) {
            callback(new Error('两次密码输入不一致'));
        }
        else {
            callback();
        }
      }
      return {
        //用于收集新增用户的对象
        addForm: {
          username: '',//用户名
          name: '',//姓名
          password: '',//密码
          repeat_password: '',//确认密码
          phone: '',//电话
          email: '',//邮箱
          is_active: false//状态
        },
        //用于收集修改用户的对象
        editForm: {
          id: '',
          name: '',
          phone: '',
          email: '',
          is_active: null,
        },
        //添加的对话框
        addDialog: false,
        //编辑的对话框
        editDialog: false,
        userList: [
          {
            username: 'zhangsan',
            name: '张三',
            phone: 110,
            email: '110@qq.com',
            create_time: '2017年11月11号',
            is_active: '激活'
          },
          {
            username: 'lisi',
            name: '李四',
            phone: 180,
            email: '110@qq.com',
            create_time: '2017年11月18号',
            is_active: '标签'
          },
          {
            username: 'laowang',
            name: '老王',
            phone: 190,
            email: '110@qq.com',
            create_time: '2017年11月13号',
            is_active: '激活'
          }
        ],
        //验证规则
        addRules: {
          username: [
            {required: true, message: '请输入用户名', trigger: 'blur'},
            {min: 3, max: 16, message: '请输入合法的用户名', trigger: 'blur'}
          ],
          name: [
            {required: true, message: '请输入姓名', trigger: 'blur'}
          ],
          password: [
            {required: true, message: '请输入密码', trigger: 'blur'},
            {min: 6, max: 12, message: '密码长度不合法', trigger: 'blur'}
          ],
          repeat_password: [
            {validator: checkPass, trigger: 'blur'}
          ],
          phone:[
//            { required: true, message: '请输入手机号', trigger: 'blur' },
//            { pattern: /^1[34578]\d{9}$/, message: '目前只支持中国大陆的手机号码' }
          ],
          email: [
            {type: 'email', required: true, message: '必须是合法邮箱格式', trigger: 'blur' },
          ]
        },
        multipleSelection: {

        },
        total: 0
      }
    },
    methods: {
      //表单提交
      submitForm: function (formName) {
        this.$refs[formName].validate((valid) => {
            if(valid) {
              //提交
              axios.post('/users/create', this.addForm).then(response => {
                  var res = response.data;
                  if(res.status == '0') {
                      //显示成功的消息提示
                      this.$message.success('新增用户成功');
                      this.resetForm('addForm');
                      //重新获取一下数据
                      this.getUsers();
                  }
                  else {
                      //返回错误的提示消息的时候
                      this.$message.error(res.message);
                  }
              }).catch(err => {
                  console.log(err);
              })
            }
            else {
              return false;
            }
        })
      },
      //清空表单
      resetForm: function (formName) {
          if(formName == 'addForm') {
            //将新增的弹出框关闭
            this.addDialog = false
          } else if(formName == 'editForm') {
              //将编辑的弹出框关闭
              this.editDialog = false;
          }
        //将弹出框里面的内容清空
        this.$refs[formName].resetFields();
      },
      getUsers: function (page) {
        axios.get('/users/getUsers', {
            params: {
                page: page || 1,
                pageSize: 5
            }
        }).then(response => {
            var res = response.data;
            this.userList = res.userList;
            this.total = res.count;
        }).catch(err => {
            console.log(err);
        })
      },
      pageChange: function (value) {
        this.getUsers(value);
      },
      //单个删除
      deleteUser: function (row) {
          this.$confirm('此操作将永久删除用户'+ row.username +', 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            axios.get('/users/delete', {
              params: {
                id: row._id
              }
            }).then(response => {
              if(response.data.message == 'success') {
                this.getUsers();
                this.$message.success('删除成功');
              }
            }).catch(err => {
              console.log(err);
            })
          }).catch(() => {
            this.$message.info

            ('已取消删除');
          });
      },
      //删除收集个数
      selectButton: function (val) {
        this.multipleSelection = val;
      },
      //删除多个
      deletesButton:function () {
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          axios.post('/users/deletes', this.multipleSelection).then(data=>{
            if (data.data.status == '0') {
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
              this.getUsers()
            }
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        })
    },
    //编辑
      setUser: function (row) {
        //编辑的弹出框开启
        this.editDialog = true;
        this.editForm._id = row._id;
        this.editForm.name = row.name;
        this.editForm.phone = row.phone;
        this.editForm.email = row.email;
        this.editForm.is_active = row.is_active;
      },
      updateUser: function () {
        axios.post('/users/updateUser', this.editForm).then(response => {
            var res = response.data;
            if(res.status == '0') {
                this.resetForm('editForm');
                this.$message.success('修改成功');
                this.getUsers();
            }
        }).catch(err => {
            console.log(err);
        })
      }
    }
  }
</script>
<style>
  h1 {
    text-align: center;
  }
  .clearfix {
    clear: both;
  }
  .fl {
    float: left;
  }
  .fr {
    float: right;
  }
  .margin40 {
    margin-top: 40px;
  }
  .block {
    margin: 20px 180px 0;
    float: right;
  }
</style>
