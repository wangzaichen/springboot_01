<template>
  <div style="padding: 10px">
    <div style="margin: 10px 0px">
      <el-button type="primary" @click="add" v-if="user.role === 1">新增</el-button>
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
        <el-table-column
            prop="name"
            label="名称">
        </el-table-column>
        <el-table-column
            prop="price"
            label="单价">
        </el-table-column>
        <el-table-column
            prop="author"
            label="作者">
        </el-table-column>
        <el-table-column
            prop="createTime"
            label="出版时间">
        </el-table-column>
        <el-table-column
            label="封面">
          <template #default="scope">
            <el-image
                style="width: 100px; height: 100px"
                :src="scope.row.cover"
                :preview-src-list="[scope.row.cover]">
            </el-image>
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
            <el-form-item label="名称">
              <el-input v-model="form.name" style="width: 80%"></el-input>
            </el-form-item>
            <el-form-item label="价格">
              <el-input v-model="form.price" style="width: 80%"></el-input>
            </el-form-item>
            <el-form-item label="作者">
              <el-input v-model="form.author" style="width: 80%"></el-input>
            </el-form-item>
            <el-form-item label="出版时间">
              <el-date-picker v-model="form.createTime" value-format="YYYY-MM-DD" type="date" style="width: 80%" clearable></el-date-picker>
            </el-form-item>
            <el-form-item label="封面">
              <el-upload ref="upload" :action="filesUploadUrl" :on-success="filesUploadSuccess">
                <el-button type="primary">点击上传</el-button>
              </el-upload>
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
  name: 'Book',
  components: {},
  data() {
    return {
      user:{},
      form:{},
      dialogVisible:false,
      search:'',
      currentPage:1,
      total:0,
      pageNum: 0,
      pageSize: 10,
      tableData: [

      ],
      filesUploadUrl: "http://localhost:9999/files/upload",
    }
  },
  created: function () {
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
    //console.log(userStr)
    // 请求服务端，确认当前登录用户的 合法信息
    request.get("/user/" + this.user.id).then(res => {
      if (res.code === '0') {
        this.user = res.data
      }
    })

    this.load()
  },

  methods:{


    filesUploadSuccess(res) {
      console.log(res)
      this.form.cover = res.data
    },

    load(){
      this.loading = true
      request.get("/book", {
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
      if (this.$refs['upload']) {
        this.$refs['upload'].clearFiles()  // 清除历史文件列表
      }
    },


    save: function () {
      //引入request
      if (this.form.id) {//更新
        request.put("/book", this.form).then(res => {
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
        request.post("/book", this.form).then(res => {
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
      request.delete("/book/" + id).then(res=>{
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
      this.$nextTick(() => {
        if (this.$refs['upload']) {
          this.$refs['upload'].clearFiles()  // 清除历史文件列表
        }
      })
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
