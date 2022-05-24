<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item
          ><i class="el-icon-lx-calendar"></i> 任务</el-breadcrumb-item
        >
        <el-breadcrumb-item>文件上传</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div>
      <el-input v-model="customer" placeholder="公司名称">
        <el-button slot="prepend" icon="el-icon-s-marketing"></el-button>
      </el-input>
    </div>
    <div class="container" align="center">
      <el-upload
        drag
        :limit="limitNum"
        :auto-upload="false"
        accept=".xlsx"
        :action="UploadUrl()"
        :before-upload="beforeUploadFile"
        :on-change="fileChange"
        :on-exceed="exceedFile"
        :on-success="handleSuccess"
        :on-error="handleError"
        :file-list="fileList"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <div class="el-upload__tip" slot="tip">
          只能上传xlsx文件，且不超过10M
        </div>
      </el-upload>
      <br />
      <el-button size="small" type="primary" @click="uploadFile"
        >立即上传</el-button
      >
      <el-button size="small">取消</el-button>
    </div>
  </div>
</template>

<script>
import VueCropper from "vue-cropperjs";
import Axios from "axios";
export default {
  name: "upload",
  data: function () {
    return {
      customer: "",
      limitNum: 1, // 上传excell时，同时允许上传的最大数
      fileList: [],
      userData: {
        id: 0,
        username: "",
        role: 0,
      },
    };
  },
  components: {
    VueCropper,
  },
  mounted() {
    this.user();
  },
  methods: {
    user() {
      this.userData.id = localStorage.getItem("ms_id");
      this.userData.username = localStorage.getItem("ms_username");
      this.userData.role = localStorage.getItem("ms_role");
    },
    // 文件超出个数限制时的钩子
    exceedFile(files, fileList) {
      this.$message.warning(
        `只能选择 ${this.limitNum} 个文件，当前共选择了 ${
          files.length + fileList.length
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
    uploadFile() {
      if (this.fileList.length === 0) {
        this.$message.warning("请上传文件");
      } else {
        let form = new FormData();
        form.append("id", this.userData.id);
        form.append("creator", this.userData.username);
        // 添加客户字段
        form.append("customer", this.customer);
        form.append("role", this.userData.role);
        form.append("uploadFiles", this.fileList[0]);
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/task/createTask",
          headers: {
            "Content-type": "multipart/form-data",
          },
          data: form,
        }).then(
          (res) => {},
          (err) => {}
        );
      }
    },
  },
};
</script>

<style scoped>
.content-title {
  font-weight: 400;
  line-height: 50px;
  margin: 10px 0;
  font-size: 22px;
  color: #1f2f3d;
}
.pre-img {
  width: 100px;
  height: 100px;
  background: #f8f8f8;
  border: 1px solid #eee;
  border-radius: 5px;
}
.crop-demo {
  display: flex;
  align-items: flex-end;
}
.crop-demo-btn {
  position: relative;
  width: 100px;
  height: 40px;
  line-height: 40px;
  padding: 0 20px;
  margin-left: 30px;
  background-color: #409eff;
  color: #fff;
  font-size: 14px;
  border-radius: 4px;
  box-sizing: border-box;
}
.crop-input {
  position: absolute;
  width: 100px;
  height: 40px;
  left: 0;
  top: 0;
  opacity: 0;
  cursor: pointer;
}
</style>