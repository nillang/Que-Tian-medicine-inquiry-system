<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 任务列表
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container" text-align="center">
      <el-table :data="tableData" border class="table" ref="multipleTable" header-cell-class-name="table-header"
        @selection-change="handleSelectionChange">
        <el-table-column width="160" prop="customer" label="客户名" align="center">
        </el-table-column>
        <el-table-column width="100" prop="creator" label="创建者" align="center">
        </el-table-column>
        <el-table-column prop="filename" label="任务名" align="center"></el-table-column>
        <el-table-column width="70" prop="total" label="总数" align="center"></el-table-column>
        <el-table-column width="90" prop="nm" label="已完成" align="center">
          <template scope="scope">
            <div @click="gotoNm(scope.row)" class="gotonm">
              {{ scope.row.nm }}
            </div>
          </template>
        </el-table-column>
        <el-table-column width="70" prop="noresult" label="无结果" align="center">
          <template scope="scope">
            <div @click="gotoNoresult(scope.row)" class="gotonoresult">
              {{ scope.row.noresult }}
            </div>
          </template>
        </el-table-column>
        <el-table-column width="80" prop="wm" label="待匹配" align="center">
          <template scope="scope">
            <div @click="gotoWm(scope.row)" class="gotowm">
              {{ scope.row.wm }}
            </div>
          </template>
        </el-table-column>
        <el-table-column width="70" prop="netcheck" label="待网查" align="center">
          <template scope="scope">
            <div @click="gotoNetcheck(scope.row)" class="gotonetcheck">
              {{ scope.row.netcheck }}
            </div>
          </template>
        </el-table-column>
        <el-table-column width="100" prop="wnetcheck" label="网查待匹配" align="center">
          <template scope="scope">
            <div @click="gotoWnetcheck(scope.row)" class="gotownetcheck">
              {{ scope.row.wnetcheck }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="time" align="center" label="修改时间"></el-table-column>
        <el-table-column label="操作" width="130" align="center">
          <template slot-scope="scope">
            <!-- <el-button
              type="text"
              icon="el-icon-mobile"
              @click="handleChoose(scope.$index, scope.row)"
              >选择</el-button
            > -->

            <el-button type="text" icon="el-icon-document" class="red" @click="handleExport(scope.$index, scope.row)">导出
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import Axios from "axios";
export default {
  name: "basetable",
  data() {
    return {
      query: {
        address: "",
        name: "",
        pageIndex: 1,
        pageSize: 10,
      },
      tableData: [],
      delList: [],
      pageTotal: 0,
      userData: {
        id: 0,
        username: "",
        role: 0,
      },
    };
  },
  mounted() {
    this.convert();
  },
  created() {
    this.user();
  },
  activated() {
    this.convert();
  },
  methods: {
    init() { },
    gotoNm(row) {
      // 判断完成字段是否为0
      if (row.nm == 0) {
        this.$message.error("没有数据");
      } else {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/finishTaskDetails",
          data: JSON.stringify({
            id: row.id,
          }),
        }).then((res) => {
          console.log(res.data);
          if (res.status == 200) {
            this.$router.push({
              path: "/taskdetail",
              query: {
                data: res.data,
              },
            });
          }
        });
      }
    },
    gotoNoresult(row) {
      // 判断无结果字段是否为0
      if (row.noresult == 0) {
        this.$message.error("没有数据");
      } else {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/cantHandleTaskDetails",
          data: JSON.stringify({
            id: row.id,
          }),
        }).then((res) => {
          console.log(res.data);
          if (res.status == 200) {
            this.$router.push({
              path: "/taskdetail",
              query: {
                data: res.data,
              },
            });
          }
        });
      }
    },
    gotoWm(row) {
      // 判断待匹配字段是否为0
      if (row.wm == 0) {
        this.$message.error("没有数据");
      } else {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/untreatedTaskDetails",
          data: JSON.stringify({
            id: row.id,
          }),
        }).then((res) => {
          if (res.status == 200) {
            this.$router.push({
              path: "/taskdetail",
              query: {
                data: res.data,
              },
            });
          }
        });
      }
    },
    gotoNetcheck(row) {
      // 判断待网查字段是否为0
      if (row.netcheck == 0) {
        this.$message.error("没有数据");
      } else {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/wnetcheckTaskDetails",
          data: JSON.stringify({
            id: row.id,
          }),
        }).then((res) => {
          console.log(res.data);
          if (res.status == 200) {
            this.$router.push({
              path: "/taskdetail",
              query: {
                data: res.data,
              },
            });
          }
        });
      }
    },
    gotoWnetcheck(row) {
      // 判断网查待匹配字段是否为0
      if (row.wnetcheck == 0) {
        this.$message.error("没有数据");
      } else {
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/netWaitMatchTaskDetails",
          data: JSON.stringify({
            id: row.id,
          }),
        }).then((res) => {
          console.log(res.data);
          if (res.status == 200) {
            this.$router.push({
              path: "/taskdetail",
              query: {
                data: res.data,
              },
            });
          }
        });
      }
    },
    user() {
      this.userData.id = localStorage.getItem("ms_id");
      this.userData.username = localStorage.getItem("ms_username");
      this.userData.role = localStorage.getItem("ms_role");
    },

    convert() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/readtask",
        data: JSON.stringify({
          id: this.userData.id,
          username: this.userData.username,
        }),
      }).then((res) => {
        this.tableData = res.data;
      });
    },

    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 选择任务
    handleChoose(index, row) {
      localStorage.setItem("ms_tid", row.id);
      localStorage.setItem("ms_tname", row.creator);
      // 跳转
      this.$router.push({
        path: "/transition",
      });
    },
    // 导出
    handleExport(index, row) {
      console.log(row.id, row.creator);
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/task/exportask",
        data: JSON.stringify({
          tid: row.id,
          customer: row.customer,
          username: row.creator,
          time: row.time,
          filename: row.filename,
        }),
        // responseType: "blob",
      }).then((res) => {
        console.log(res);
        if (res.status == 200) {
          const filename = res.headers.filename
          console.log(filename);
          // 下载文件
          // 假设 data 是返回来的二进制数据
          const data = res.data
          const url = window.URL.createObjectURL(new Blob([data], { type: 'application/vnd.ms-excel'}))
          const link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', filename)
          document.body.appendChild(link)
          link.click()
        }
      });
    },
  },
};
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
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
</style>
