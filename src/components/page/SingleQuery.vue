<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 单条查询
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <div id="handle-l">
          <el-select size="mini" v-model="select.searchValue" multiple placeholder="请选择" @change="chickvalue">
            <el-option v-for="item in select.options" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          <el-input v-model="query.username" placeholder="地址名称" class="handle-input-u mr10"></el-input>
          <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>

          <!-- <el-button type="primary" icon="el-icon-search" @click="test">测试</el-button> -->

          <el-button type="primary" @click="dialogVisible = true">上传<i class="el-icon-upload el-icon--right"></i>
          </el-button>
          <el-dialog title="上传任务" :visible.sync="dialogVisible" width="50%">
            <div class="container" align="center">
              <el-select size="mini" v-model="select.searchValue" multiple placeholder="请选择客户" @change="chickvalue"
                style="width: 59%">
                <el-option v-for="item in select.options" :key="item.value" :label="item.label" :value="item.value">
                </el-option>
              </el-select>
              <el-upload drag :limit="limitNum" :auto-upload="false" accept=".xlsx" :action="UploadUrl()"
                :before-upload="beforeUploadFile" :on-change="fileChange" :on-exceed="exceedFile"
                :on-success="handleSuccess" :on-error="handleError" :file-list="fileList">
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">
                  将文件拖到此处，或<em>点击上传</em>
                </div>
                <div class="el-upload__tip" slot="tip">
                  只能上传xlsx文件，且不超过10M
                </div>
              </el-upload>
              <br />
              <el-button size="small" type="primary" @click="uploadFileOfTask">立即上传</el-button>
              <el-button size="small">取消</el-button>
            </div>
          </el-dialog>
        </div>
        <div id="handle-r">
          <el-button v-show="isEmptyObject()" icon="el-icon-delete" class="red" id="unbind" @click="unbind">解除绑定
          </el-button>
          <span v-show="istypeof()">
            模糊匹配的条数为：{{ paging.dataLength }}</span>
        </div>
      </div>
      <div>
        <el-table v-if="isEmptyObject()" :data="pv" style="width: 100%">
          <el-table-column type="expand">
            <template slot-scope="props">
              <el-form label-position="left" inline class="demo-table-expand">
                <el-form-item label="ID:">
                  <span>{{ props.row.id }}</span><br />
                </el-form-item>
                <el-form-item label="客户:">
                  <span>{{ props.row.customer }}</span><br />
                </el-form-item>
                <el-form-item label="经销商编码:">
                  <span>{{ props.row.dc }}</span><br />
                </el-form-item>
                <el-form-item label="经销商名称:">
                  <span>{{ props.row.dn }}</span>
                </el-form-item>
                <el-form-item label="买方名称:">
                  <span>{{ props.row.bn }}</span>
                </el-form-item>
                <el-form-item label="标准客户编码">
                  <span>{{ props.row.scc }}</span>
                </el-form-item>
                <el-form-item label="标准客户名称:">
                  <span>{{ props.row.scn }}</span>
                </el-form-item>
                <el-form-item label="上次更新是:">
                  <span>{{ props.row.user }}</span>
                </el-form-item>
                <el-form-item label="修改时间:">
                  <span>{{ props.row.time }}</span>
                </el-form-item>
              </el-form>
            </template>
          </el-table-column>
          <el-table-column prop="scn" label="查询成功"> </el-table-column>
        </el-table>

        <el-table :data="vague" v-if="vague.length > 0" border class="table" header-cell-class-name="table-header">
          <el-table-column prop="customer" label="客户" width="110" align="center">
          </el-table-column>
          <el-table-column prop="dc" label="经销商编码" width="100" align="center">
            <template slot-scope="scope">{{ scope.row.dc ? scope.row.dc : "--" }}</template>
          </el-table-column>
          <el-table-column prop="dn" label="经销商名称" width="200" align="center">
            <template slot-scope="scope">{{ scope.row.dn ? scope.row.dn : "--" }}</template>
          </el-table-column>
          <el-table-column prop="bn" label="买方名称" width="150" align="center">
            <template slot-scope="scope">{{ scope.row.bn ? scope.row.bn : query.username }}</template>
          </el-table-column>
          <el-table-column prop="scc" label="标准客户编码" width="110" align="center">
          </el-table-column>
          <el-table-column prop="scn" label="标准客户名称" width="283" align="center">
          </el-table-column>
          <el-table-column prop="time" label="修改时间" width="160" align="center">
          </el-table-column>
          <el-table-column label="操作" width="120" align="center">
            <template slot-scope="scope">
              <el-button type="text" icon="el-icon-circle-check" class="red"
                @click="handleInsert(scope.$index, scope.row)">绑定</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination v-if="istypeof()" background :current-page.sync="paging.currentPage"
          @current-change="handlePageChange" layout="prev, pager, next" :total="paging.pageTotal" align="center">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import Axios from "axios";
export default {
  name: "basetable",
  data() {
    return {
      limitNum: 1, // 上传excell时，同时允许上传的最大数
      fileList: [],

      dialogVisible: false,
      form: {
        name: '',
        number: ''
      },

      query: {
        id: 0,
        customername: "",
        username: "",
        user: "",
      },
      select: {
        searchValue: {},
        options: [],
      },
      paging: {
        dataLength: 0,
        currentPage: 1,
        pageSize: 5,
        pageTotal: 0,
        pageData: [], //分页总数据
      },
      perfect: {},
      pv: [],
      vague: [],
    };
  },
  created() {
    this.convert();
  },
  methods: {
    test() {
      this.$router.push({
        path: "/task",
      });
    },
    uploadFileOfTask() {
      if (this.fileList.length === 0) {
        this.$message.warning("请上传文件");
      } else {
        let form = new FormData();
        form.append("id", localStorage.getItem("ms_id"));
        form.append("customer", this.query.customername);
        form.append("creator", localStorage.getItem("ms_username"));
        // 添加客户字段
        form.append("uploadFiles", this.fileList[0]);
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/upload/uploadTask",
          headers: {
            "Content-type": "multipart/form-data",
          },
          data: form,
        }).then(
          (res) => {
            // 提示上传成功
            this.$message({
              message: "上传成功",
              type: "success",
            });
            // 跳转到task页面
            this.$router.push({
              path: "/task",
            });
          },
          (err) => {
            this.$message.error("上传失败");
          }
        );
      }
    },
    //获取下拉框数据
    chickvalue(val) {
      var name = "";
      for (let i = 0; i <= val.length - 1; i++) {
        this.select.options.find((item) => {
          if (item.value == val[i]) {
            name += item.label;
            this.query.id = item.value + "";
          }
        });
      }
      this.query.customername = name;
    },
    convert() {
      Axios({
        method: "get",
        url: "http://127.0.0.1:3305/querySystem/upload/getCustomer",
      }).then((res) => {
        for (const key in res.data) {
          this.select.options.push({ value: 0, label: "" });
          this.select.options[key].value = res.data[key].id;
          this.select.options[key].label = res.data[key].customer;
        }
      });
    },
    // 解除绑定
    unbind() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/single/unbind",
        data: JSON.stringify({
          id: this.pv[0].id,
          bn: this.pv[0].bn,
        }),
      }).then((res) => {
        if (res.status == 200) {
          this.$message({
            message: "解绑成功",
            type: "success",
          });
          this.handleSearch();
        } else {
          this.$message({
            message: "解绑失败",
            type: "error",
          });
        }
      });
    },
    add() {
      this.pv.push(this.perfect);
    },
    istypeof() {
      if (this.vague.length > 0) {
        return true;
      } else {
        return false;
      }
    },
    isEmptyObject() {
      if (this.perfect.hasOwnProperty("id")) {
        return true;
      }
      return false;
    },
    // 触发搜索按钮
    handleSearch() {
      this.pv = [];
      this.vague = [];
      Axios({
        method: "post",
        url: "http://localhost:3305/querySystem/single/searchQuery",
        data: JSON.stringify({
          bn: this.query.username,
          CustomerName: this.query.customername,
        }),
      }).then((res) => {
        if (res.status == 200) {
          if (res.data.perfect.bn != "") {
            this.perfect = res.data.perfect.result;
            this.vague = [];
            this.add();
          } else {
            console.log(res.data);
            this.paging.pageData = res.data.vagueMap.result;
            this.paging.pageData = this.paging.pageData.concat(
              res.data.vagueMaster.result
            );
            this.paging.dataLength = this.paging.pageData.length;
            this.paging.pageTotal =
              Math.ceil(this.paging.dataLength / this.paging.pageSize) * 10;
            this.vague = this.paging.pageData.slice(
              0,
              this.paging.pageSize
            );
            console.log(this.vague);
            this.perfect = {};
          }
        }
      });
    },

    handleSearchBatch() {
      // 跳转到任务列表页面
      this.$router.push({
        path: "/add",
      });
    },
    // 确认绑定
    handleInsert(index, row) {
      console.log(index, row);
      this.$confirm("再次确认绑定", "提示", {
        type: "warning",
      }).then(() => {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/single/mnualDetermination",
          data: JSON.stringify({
            customer: row.customer,
            dc: row.dc,
            dn: row.dn,
            bn: this.query.username,
            scc: row.scc,
            scn: row.scn,
            user: localStorage.getItem("ms_username"),
          }),
        }).then((res) => {
          if (res.status == 200) {
            this.$message({
              message: "绑定成功",
              type: "success",
            });
            this.handleSearch();
          }
        });
      });
    },
    // 分页导航
    handlePageChange(val) {
      var start = (val - 1) * this.paging.pageSize;
      var end = start + this.paging.pageSize;
      this.vague = this.paging.pageData.slice(start, end);
      // this.$set(this.query, "pageIndex", val);
      // this.getData();
    },
    // 文件超出个数限制时的钩子
    exceedFile(files, fileList) {
      this.$message.warning(
        `只能选择 ${this.limitNum} 个文件，当前共选择了 ${files.length + fileList.length
        } 个`
      );
    },
    // 文件状态改变时的钩子
    fileChange(file, fileList) {
      console.log(file.raw);
      this.fileList.push(file.raw);
      console.log(this.fileList);
    },
    // 上传文件之前的钩子, 参数为上传的文件,若返回 false 或者返回 Promise 且被 reject，则停止上传
    beforeUploadFile(file) {
      console.log("before upload");
      console.log(file);
      let extension = file.name.substring(file.name.lastIndexOf(".") + 1);
      let size = file.size / 1024 / 1024;
      if (extension !== "xlsx") {
        this.$message.warning("只能上传后缀是.xlsx的文件");
      }
      if (size > 10) {
        this.$message.warning("文件大小不得超过10M");
      }
    },
    // 文件上传成功时的钩子
    handleSuccess(res, file, fileList) {
      this.$message.success("文件上传成功");
    },
    // 文件上传失败时的钩子
    handleError(err, file, fileList) {
      this.$message.error("文件上传失败");
    },
    UploadUrl: function () {
      // 因为action参数是必填项，我们使用二次确认进行文件上传时，直接填上传文件的url会因为没有参数导致api报404，所以这里将action设置为一个返回为空的方法就行，避免抛错
      return "";
    },
  },

};
</script>

<style scoped lang="less">
.handle-box {
  display: flex;
  margin-bottom: 20px;
}

#handle-l {
  width: 70%;
}

#handle-r {
  width: 30%;
}

#handle-r>span {
  display: block;
  text-align: right;
  margin-top: 5px;
  margin-right: 1px;
}

#unbind {
  margin-left: 72.5%;
}

.handle-input-u {
  margin-left: 10px;
  width: 280px;
  display: inline-block;
}

.table {
  width: 100%;
  font-size: 14px;
}

.red {
  color: #ff0000;
}

.mr10 {
  margin-right: 10px;
}

.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}

.demo-table-expand {
  font-size: 0;
}

.demo-table-expand label {
  width: 100%;
  color: #99a9bf;
}

.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  padding: 0 5%;
  width: 70%;
}

.button_batch {
  border-color: #d6d31a;
  background-color: #d6d31a;
}

.splitLine {
  //黑色加粗
  height: 5px;
  background-color: #99a9bf;
  // 行高
  margin: 10px 0 10px 0;
  // 弧度
  border-radius: 5px;
}
</style>
