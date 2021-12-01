<template>
  <div style="padding: 10px">
    <div style="margin: 10px 0px">
      <el-button type="primary" @click="add">新增</el-button>
      <el-button type="primary">导入</el-button>

      <el-button type="primary">导出</el-button>

    </div >
    <div >
      <!--    搜索区域-->
      <div style="margin: 10px 0px; width: 60%">
        <el-input v-model="search" placeholder="请输入关键字" style="margin: 10px 5px; width: 35%" clearable></el-input>
        <el-button type="primary" style="margin: 10px 0px; width: 15%" @click = "load">查询</el-button>
      </div>

      <el-table :data="tableData" stripe style=" width:1600px">
        <el-table-column prop="id" label="ID" > </el-table-column>
        <el-table-column prop="username" label="用户名" > </el-table-column>
        <el-table-column prop="nickName" label="昵称"> </el-table-column>

        <el-table-column prop="age" label="年龄" > </el-table-column>
        <el-table-column prop="sex " label="性别" > </el-table-column>
        <el-table-column prop="address" label="地址"> </el-table-column>
        <el-table-column
            label="角色">
          <template #default="scope">
            <span v-if="scope.row.role === 1">管理员</span>
            <span v-if="scope.row.role === 0">普通用户</span>
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="100">
          <template #default="scope">
            <el-button size="mini" @click="handleEdit(scope.row)" type="text" >编辑</el-button>

            <el-popconfirm title="这是一段内容确定删除吗？" @confirm="handleDelete(scope.row.id)">
              <template #reference>
                <el-button type="text" size="small">删除</el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>


      <div>
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-sizes="[5, 10, 20]"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
        >
        </el-pagination>

        <el-dialog
            v-model="dialogVisible"
            title="提示"
            width="30%">
          <el-form :model="form" label-width="120px">
            <el-form-item label="用户名" style="width: 80%">
              <el-input v-model="form.username"></el-input>
            </el-form-item>
            <el-form-item label="昵称" style="width: 80%">
              <el-input v-model="form.nickName"></el-input>
            </el-form-item>
            <el-form-item label="年龄" style="width: 80%">
              <el-input v-model="form.age"></el-input>
            </el-form-item>
            <el-form-item label="性别" style="width: 80%">
              <el-radio v-model="form.sex" label="男">男</el-radio>
              <el-radio v-model="form.sex" label="女">女</el-radio>
              <el-radio v-model="form.sex" label="未知">未知</el-radio>
            </el-form-item>
            <el-form-item label="地址" style="width: 80%">
              <el-input type="textarea" v-model="form.address"></el-input>
            </el-form-item>
          </el-form>

          <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="save">确认</el-button>
      </span>
          </template>
        </el-dialog>
      </div>

    </div>



  </div>
</template>

<script>
// @ is an alias to /src




import request from "@/utills/request";

export default {
  name: 'Home',
  components: {},
  data() {
    return {
      form:{},
      dialogVisible:false,
      search:'',
      currentPage:1,
      total:0,
      pageNum: 0,
      pageSize: 10,
      tableData: [

      ],
    }
  },
  created() {
    this.load()
    },

  methods:{
    load(){
      this.loading = true
      request.get("/user", {
       params:{
         pageNum: this.currentPage,
         pageSize: this.pageSize,
         search: this.search
       }
      }).then(res => {
        //console.log(res)
        this.loading = false
        this.tableData = res.data.records
        this.total = res.data.total
      })
    },
    add(){
      this.dialogVisible = true
      this.form = {}
    },


    save: function () {
      //引入request
      if (this.form.id) {//更新
        request.put("/user", this.form).then(res => {
          //console.log(res)
          if (res.code === '0') {
            this.$message({
              type: "suceess",
              message: "更新成功"
            })
          } else {
            this.$message({
              type: "error",
              message: req.msg
            })
          }

          //刷新表格数据
          this.load()
          this.dialogVisible = false // 关闭弹窗
        })
      } else {//新增
        request.post("/user", this.form).then(res => {
          //console.log(res)
          if (res.code === '0') {
            this.$message({
              type: "suceess",
              message: "新增成功"
            })
          } else {
            this.$message({
              type: "error",
              message: req.msg
            })
          }

          //刷新表格数据
          this.load()
          this.dialogVisible = false // 关闭弹窗
        })
      }


    },

    handleDelete(id){
      //console.log(id)
      request.delete("/user/" + id).then(res=>{
        if (res.code === '0') {
          this.$message({
            type: "suceess",
            message: "删除成功"
          })
        } else {
          this.$message({
            type: "error",
            message: req.msg
          })
        }

        //刷新表格数据
        this.load()
      })

    },

    //如何将深拷贝 浅拷贝
    handleEdit(row){
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true

    },

    //改变当前页面个数触发
    handleSizeChange(pageSize){
      this.pageSize = pageSize;
      this.load()
    },

    handleCurrentChange(pageNum){
      this.currentPage  = pageNum;
      this.load()
    }
  }
}
</script>
