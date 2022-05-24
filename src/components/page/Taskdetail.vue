<template>
  <div>
    <!-- 显示跳转传递的数据 -->
    <el-badge v-show="LimitShow()" class="top_button">
      <el-button @click="gotoHandle" size="small">去处理</el-button>
    </el-badge>
    <el-table :data="tableData" border class="table" ref="multipleTable" header-cell-class-name="table-header">
      <el-table-column width="160" prop="username" label="修改人" align="center"></el-table-column>
      <el-table-column width="225" prop="dc" label="经销商编码" align="center"></el-table-column>
      <el-table-column width="370" prop="dn" label="经销商名称" align="center"></el-table-column>
      <el-table-column width="370" prop="bn" label="买方名称" align="center"></el-table-column>
      <el-table-column width="170" prop="time" label="修改时间" align="center"></el-table-column>
    </el-table>
    <div class="pagination">
      <el-pagination background :current-page.sync="paging.currentPage" @current-change="handlePageChange"
        layout="prev, pager, next" :total="paging.pageTotal">
      </el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  name: "taskdetail",
  data() {
    return {
      state: 0,
      tableData: [],
      paging: {
        dataLength: 0,
        currentPage: 1,
        pageSize: 11,
        pageTotal: 0,
        pageData: [], //分页总数据
      },
    };
  },
  created() {
    this.getData();
  },
  activated() {
    this.getData();
  },
  methods: {
    gotoHandle() {
      if (this.state == 0) {
        this.$router.push({
          path: "/transition",
        });
      } else {
        this.$router.push({
          path: "/transition",
        });
      }
    },

    // 获取跳转的数据，并赋值给当前页面, 再获取分页数据
    getData() {
      this.paging.pageData = this.$route.query.data;
      this.state = this.paging.pageData[0].state;
      this.paging.dataLength = this.paging.pageData.length;
      this.paging.pageTotal =
        Math.ceil(this.paging.dataLength / this.paging.pageSize) * 10;
      this.tableData = this.paging.pageData.slice(0, this.paging.pageSize);
    },

    // 当要处理待匹配和待网查的数据时显示
    LimitShow() {
      if (this.tableData[0].state == 0 || this.tableData[0].state == 4) {
        return true;
      }
      return false;
    },

    // 分页
    handlePageChange(val) {
      var start = (val - 1) * this.paging.pageSize;
      var end = start + this.paging.pageSize;
      this.tableData = this.paging.pageData.slice(start, end);
      // this.$set(this.query, "pageIndex", val);
      // this.getData();
    },
  },
  beforeRouteEnter(to, from, next) {
    console.log(to);
    console.log(from);
    if (from.name == 'Index') {
      to.meta.isFresh = true;
    }
    next();
  }
};
</script>

<style scoped lang="less">
.top_button {
  margin-left: 94.8%;
}
</style>