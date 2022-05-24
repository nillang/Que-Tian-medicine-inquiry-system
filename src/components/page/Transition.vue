<template>
  <div>
    <div class="hander">
      <span>{{ taskDails.taskDails.dn }}---{{ taskDails.taskDails.bn }}</span>
      <!-- <el-button class="button" type="primary" icon="el-icon-search" @click="getMap">搜索</el-button> -->
      <el-button class="button_cancelProcess" type="primary" icon="el-icon-search" @click="cancelProcess">无法解决
      </el-button>
    </div>
    <div>
      <el-table :data="vague" border class="table" header-cell-class-name="table-header">
        <el-table-column prop="id" label="ID" width="120" align="center">
        </el-table-column>
        <el-table-column prop="customer" label="客户" width="210" align="center">
        </el-table-column>
        <el-table-column prop="scc" label="标准客户编码" width="210" align="center">
        </el-table-column>
        <el-table-column prop="scn" label="标准客户名称" width="315" align="center">
        </el-table-column>
        <el-table-column prop="time" label="修改时间" width="260" align="center">
        </el-table-column>
        <el-table-column label="操作" width="180" align="center">
          <template slot-scope="scope">
            <el-button type="text" icon="el-icon-circle-check" class="red"
              @click="handleInsertMap(scope.$index, scope.row)">确认</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination background :current-page.sync="paging.currentPage" @current-change="handlePageChange"
          layout="prev, pager, next" :total="paging.pageTotal">
        </el-pagination>
      </div>
    </div>
    <div>
      <el-table :data="taskDails.netcheck.Result" border class="table" header-cell-class-name="table-header">
        <!-- <el-table-column prop="KeyNo" label="KeyNo" width="120" align="center">
          </el-table-column> -->
        <el-table-column prop="Name" label="企业名称" width="275" align="center">
        </el-table-column>
        <el-table-column prop="CreditCode" label="统一社会信用代码" width="170" align="center">
        </el-table-column>
        <el-table-column prop="StartDate" label="成立日期" width="120" align="center">
        </el-table-column>
        <el-table-column prop="OperName" label="法定代表人姓名" width="120" align="center">
        </el-table-column>
        <el-table-column prop="Status" label="状态" width="120" align="center">
        </el-table-column>
        <el-table-column prop="No" label="注册号" width="120" align="center">
        </el-table-column>
        <el-table-column prop="Address" label="注册地址" width="370" align="center">
        </el-table-column>
        <el-table-column label="操作" width="120" align="center">
          <template slot-scope="scope">
            <el-button type="text" icon="el-icon-circle-check" class="red"
              @click="handleInsertNet(scope.$index, scope.row)">确认</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- <div style="margin-top: 20px">

      
    </div> -->
  </div>
</template>

<script>
import Vue from "vue";
import Axios from "axios";
export default {
  name: "transition",
  data() {
    return {
      // 接受Task.vue传来的数据
      activeName: "second",
      tid: 0,
      tname: "",
      taskDails: {
        taskDails: {
          id: 0,
          tid: 0,
          customer: "",
          username: "",
          dc: "",
          dn: "",
          bn: "",
          state: 0,
          time: "",
        },
        perfect: [],
        netcheck: {
          Paging: {
            PageSize: 0,
            PageIndex: 0,
            TotalRecords: 0,
          },
          Result: [],
          Status: "",
          Message: "",
          OrderNumber: "",
        },
      },
      paging: {
        pageSize: 2, //每页显示条数
        dataLength: 0,
        currentPage: 1, //当前页
        pageTotal: 0, //总页数
        pageData: [], //分页总数据
      },
      vague: [],
      network: [],
      id: -1,
    };
  },
  mounted() {
    window.addEventListener("beforeunload", (e) => this.beforeunloadFn(e));
  },
  destroyed() {
    window.removeEventListener("beforeunload", (e) => this.beforeunloadFn(e));
  },
  created() {
    this.getdata();
    this.getTaskDails();
  },
  methods: {
    beforeunloadFn(e) {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/releasetaskdetails",
        data: JSON.stringify({
          id: this.taskDails.id,
        }),
      }).then((res) => {
        console.log("transition释放成功");
      });
    },

    // 获取task数据
    getdata() {
      this.tid = localStorage.getItem("ms_tid");
      this.tname = localStorage.getItem("ms_tname");
    },
    getTaskDails() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/batch/batchhandle",
        data: JSON.stringify({
          tid: this.tid,
          username: this.tname,
        }),
      }).then((res) => {
        console.log(res.data);
        this.taskDails = res.data;
        localStorage.setItem("ms_tdid", this.taskDails.taskDails.id);
        this.viewProcessedData();
      });
    },
    // 查看已处理的数据
    viewProcessedData() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/readtaskdetailsByUser",
        data: JSON.stringify({
          id: this.tid,
          username: localStorage.getItem("ms_username"),
        }),
      }).then((res) => {
        this.processedData = res.data;
      });
    },
    // batch() {
    //   console.log(this.taskDails.bn);
    //   console.log(this.taskDails.customer);
    //   Axios({
    //     method: "post",
    //     url: "http://127.0.0.1:3305/querySystem/batch/batchQueryInMap",
    //     data: JSON.stringify({
    //       bn: this.taskDails.bn,
    //       CustomerName: this.taskDails.customer,
    //     }),
    //   }).then((res) => {
    //     var data = res.data;
    //     console.log(data.mapdata === null);
    //     if (data.mapdata === null) {
    //       // 提示请网查
    //       this.$message({
    //           message: "请网查",
    //           type: "success",
    //         });
    //     } else {
    //       this.paging.dataLength = data.mapdata.length;
    //       this.paging.pageData = data.mapdata;
    //       this.paging.pageTotal =
    //         Math.ceil(this.paging.pageData.length / this.paging.pageSize) * 10;
    //       this.vague = this.paging.pageData.slice(0, this.paging.pageSize);
    //     }
    //   });
    // },
    handleInsertMap(index, row) {
      console.log(row.id);
      console.log(row.CustomerName);
      console.log(row.scn);
      return
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/batch/batchcompare",
        data: JSON.stringify({
          customerName: row.CustomerName,
          bn: row.scn,
        }),
      }).then((res) => {
        if (res.status == 200) {
          this.$message({
            message: "绑定成功",
            type: "success",
          });
          //重新加载页面
          this.getTaskDails();
          this.pageData = [];
        }
      });
    },
    handleInsertNet(index, row) {
      console.log(row.id);
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/batchmnualDetermination",
        data: JSON.stringify({
          customerName: this.taskDails.taskDails.customer,
          bn: row.Name,
        }),
      }).then((res) => {
        if (res.status == 200) {
          this.$message({
            message: "绑定成功",
            type: "success",
          });
          //重新加载页面
          this.getTaskDails();
          this.pageData = [];
        }
      });
    },
    handleInsert(index, row) {
      console.log(row.id);
      this.$confirm("再次确认绑定", "提示", {
        type: "warning",
      }).then(() => {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/batchmnualDetermination",
          data: JSON.stringify({
            tid: this.taskDails.tid,
            tdid: this.taskDails.id,
            bn: this.taskDails.bn,
            customer: this.taskDails.customer,
            dc: this.taskDails.dc,
            dn: this.taskDails.dn,
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
            //重新加载页面
            this.getTaskDails();
            this.pageData = [];
          }
        });
      });
    },
    cancelProcess() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/cancelProcess",
        data: JSON.stringify({
          id: this.taskDails.id,
          tid: this.taskDails.tid,
          lastname: this.taskDails.username,
          nowname: localStorage.getItem("ms_username"),
        }),
      }).then((res) => {
        // 提示操作成功
        this.$message({
          message: "取消成功",
          type: "success",
        });
        // 重新加载页面
        this.getTaskDails();
        this.pageData = [];
      });
    },
    // 分页导航
    handlePageChange(val) {
      var start = (val - 1) * this.paging.pageSize;
      var end = start + this.paging.pageSize;
      this.vague = this.pageData.slice(start, end);
      // this.$set(this.paging, "pageIndex", val);
      // this.getData();
    },
    taskdetail() {
      localStorage.setItem("ms_match", JSON.stringify(this.match));
      this.$router.push({
        path: "/taskdetail",
      });
    },
  },
};
</script>

<style lang="less" scoped>
.box-card {
  width: 50%;
}

.box {
  display: flex;
}

.text {
  font-size: 10px;
}

.item {
  margin-bottom: 20px;
}

.item>span {
  display: block;
  line-height: 25px;
}

.button {
  margin-left: 5%;
}

.hander {
  width: 100%;
  height: 5%;
}

.hander>span {
  width: 20%;
}

.button_cancelProcess {
  margin-left: 63%;
  border-color: red;
  background-color: red;
}
</style>