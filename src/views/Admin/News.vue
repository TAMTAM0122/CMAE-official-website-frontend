<template>
  <div class="news">
    <el-form :model="QueryData" inline>
    <el-button type="primary" @click="openDialog()">新增新聞</el-button>
    <el-form-item label="">
        <el-input v-model="QueryData.title" placeholder="请输入新闻标题"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleQuery">查询</el-button>
        <el-button @click="resetForm">重置</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="tableData" border style="width: 100%" v-loading="loading">
      <el-table-column prop="id" label="序号" width="180"></el-table-column>
      <el-table-column prop="title" label="新闻标题" width="180"></el-table-column>
      <el-table-column prop="img" label="图片">
        <template slot-scope="scope">
          <img style="width:100%" :src="imgserver + scope.row.img" alt />
        </template>
      </el-table-column>
      <el-table-column prop="content" label="新闻内容">
        <template slot-scope="scope">
          <p v-if="scope.row.content.length > 100">{{scope.row.content.substring(0,100)}} ...</p>
          <p v-else>{{scope.row.content}}</p>
        </template>
      </el-table-column>
      <el-table-column prop="type" label="新闻类别">
        <template slot-scope="scope">{{scope.row.type == 1 ? '公司新闻':'行业动态'}}</template>
      </el-table-column>
      <el-table-column prop="eventTime" label="事件时间" width="180"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            @click="handleEdit(scope.$index, scope.row)"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="handleDelete(scope.$index, scope.row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--  -->
    <el-dialog title="新闻编辑" :visible.sync="dialogFormVisible">
      <el-form :model="formData">
        <el-form-item label="新闻名称" :label-width="formLabelWidth">
          <el-input v-model="formData.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="事件时间" :label-width="formLabelWidth">
            <el-date-picker
              v-model="formData.eventTime"
              type="datetime"
              placeholder="选择日期时间">
            </el-date-picker>
        </el-form-item>
        <el-form-item label="新闻图片" :label-width="formLabelWidth">
          <el-upload
            class="avatar-uploader"
            action="http://localhost:8081/Tool/Upload?type=news"
            :headers="headers"
            :show-file-list="false"
            :on-success="handleSuccess"
          >
            <img v-if="formData.img" :src="imgserver + formData.img" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="新闻内容" :label-width="formLabelWidth">
          <el-input type="textarea" :rows="10" v-model="formData.content" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="新闻类别" :label-width="formLabelWidth">
          <el-radio v-model="formData.type" :label="1">公司新闻</el-radio>
          <el-radio v-model="formData.type" :label="2">行业动态</el-radio>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleCreateOrModify()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "loginNews",
  data() {
    return {
      options: {},
      headers: {},
      tableData: [],
      QueryData: { title: '' },  
      formData: {
        id: 0,
        title: "",
        img: "",
        type: 1,
        content: "",
        eventTime: "",
        createTime: new Date()
      },
      dialogFormVisible: false,
      formLabelWidth: "120px",
      loading: true
    };
  },
  mounted() {
    let token = "Bearer " + sessionStorage.getItem("token");
    //window.console.log(token);
    this.options = {
      headers: {
        Authorization: token
      }
    };
    this.headers = {
      Authorization: token
    };

    this.loadData();
  },
  methods: {
    handleQuery() {
        const queryObject = this.QueryData;  // 直接使用对象
        console.log(queryObject);  
        this.$http
        .post("News/FindNewsByTitle", queryObject)
        .then(response => {
            // 假设后端返回查询结果
            this.tableData = response.data;
        })
        .catch(error => {
          console.error('查询失败:', error);
        });
    },
    resetForm() {
      this.QueryData = {
        job: ''
      };
      this.QueryData = []; // 重置表格数据
    },
    handleSuccess(response, file, fileList) {
      window.console.log(response, file, fileList);
      this.formData.img = response;
    },
    loadData() {
      this.loading = true;
      this.$http
        .get("News/GetNewsAll?type=0&num=10")
        .then(response => {
          // window.console.log(response);
          this.tableData = response.data;
          this.loading = false;
        })
        .catch(e => {
          this.$message({
            message: "网络或程序异常！" + e,
            type: "error"
          });
        });
    },
    openDialog() {
      // 清除数据
      this.formData.id = 0;
      this.formData.title = "";
      this.formData.img = "";
      this.formData.type = 1;
      this.formData.content = "";
      this.formData.eventTime = "";
      this.formData.createTime = new Date();

      this.dialogFormVisible = true;
    },
    handleCreateOrModify() {
      if (!this.formData.id) {
        this.loading = true;
        this.$http
          .post("News/CreateNews", this.formData, this.options)
          .then(response => {
            window.console.log(response);
            this.loading = false;
            this.$message({
              message: "创建成功！",
              type: "success"
            });
            this.dialogFormVisible = false;
            this.loadData();
          })
          .catch(e => {
            this.$message({
              message: "网络或程序异常！" + e,
              type: "error"
            });
          });
      } else {
        this.loading = true;
        this.$http
          .post("News/ModifiedNews", this.formData, this.options)
          .then(response => {
            this.loading = false;
            window.console.log(response);
            this.$message({
              message: "修改成功！",
              type: "success"
            });
            this.dialogFormVisible = false;
            this.loadData();
          })
          .catch(e => {
            this.$message({
              message: "网络或程序异常！" + e,
              type: "error"
            });
          });
      }
    },
    //编辑
    handleEdit(index, row) {
      //index:第几行   row:这一行的数据
      window.console.log(index, row);
      // this.formData = row;
      this.formData = JSON.parse(JSON.stringify(row));
      this.formData.type = Number(this.formData.type);
      this.dialogFormVisible = true;
    },
    handleDelete(index, row) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          // 已确认删除
          // 调接口删除
          this.loading = true;
          this.$http
            .post(`News/DeleteNews?id=${row.id}`, null, this.options)
            .then(response => {
              this.loading = false;
              window.console.log(response);
              this.$message({
                message: "删除成功！",
                type: "success"
              });
              this.loadData();
            })
            .catch(e => {
              this.$message({
                message: "网络或程序异常！" + e,
                type: "error"
              });
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.el-table {
  margin-top: 20px;
}
</style>