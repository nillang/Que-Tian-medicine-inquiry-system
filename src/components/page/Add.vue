<template>
  <div class="body">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 添加/上传</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div>
      <el-collapse v-model="activeName" accordion>
        <el-collapse-item title="上传客户名称    Upload customer name" name="1">
          <div class="addCustomer">
            <el-input v-model="addCustomer.customer" placeholder="公司名称" class="input">
              <el-button slot="prepend" icon="el-icon-s-marketing">请输入</el-button>
            </el-input>
            <el-button class="button" type="primary" @click="uploadCustomer">立即上传</el-button>
          </div>
        </el-collapse-item>
        <el-collapse-item title="上传主数据     Upload master data" name="2">
          <!-- <div>
          </div> -->
          <div class="container" align="center">
            <el-select size="mini" v-model="select.searchValue" multiple placeholder="请选择客户" @change="chickvalue"
              style="width: 29%">
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
            <el-button size="small" type="primary" @click="uploadFileOfMaster">立即上传</el-button>
            <el-button size="small">取消</el-button>
          </div>
        </el-collapse-item>
        <el-collapse-item title="上传map关系表  Upload map relation table" name="3">
          <div class="container" align="center">
            <el-select size="mini" v-model="select.searchValue" multiple placeholder="请选择客户" @change="chickvalue"
              style="width: 29%">
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
            <el-button size="small" type="primary" @click="uploadFileOfMap">立即上传</el-button>
            <el-button size="small">取消</el-button>
          </div>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>

<script>
import VueCropper from "vue-cropperjs";
import Axios from "axios";
export default {
  name: "add",
  data() {
    return {
      limitNum: 1, // 上传excell时，同时允许上传的最大数
      fileList: [],
      userData: {
        id: 0,
        username: "",
      },
      addCustomer: {
        customer: "",
      },
      addMap: {
        customer: "",
        limitNum: 1, // 上传excell时，同时允许上传的最大数
        fileList: [],
      },
      activeName: "2",
      select: {
        id: 0,
        options: [],
        searchValue: "",
        selectValue: {},
      },
    };
  },
  components: {
    VueCropper,
  },
  mounted() {
    this.user();
  },
  created() {
    this.convert();
  },
  methods: {
    user() {
      this.userData.id = localStorage.getItem("ms_id");
      this.userData.username = localStorage.getItem("ms_username");
    },

    uploadFileOfMap() {
      if (this.fileList.length === 0) {
        this.$message.warning("请上传文件");
      } else {
        let form = new FormData();
        form.append("creator", this.userData.username);
        // 添加客户字段
        form.append("customer", this.addMap.customer);
        form.append("uploadFiles", this.fileList[0]);
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/upload/uploadMap",
          headers: {
            "Content-type": "multipart/form-data",
          },
          data: form,
        }).then(
          (res) => {
            this.$message({
              message: "上传成功",
              type: "success",
            });
          },
          (err) => {
            this.$message.error("上传失败");
          }
        );
      }
    },

    uploadFileOfMaster() {
      if (this.fileList.length === 0) {
        this.$message.warning("请上传文件");
      } else {
        let form = new FormData();
        form.append("customer", this.addCustomer.customer);
        // 添加客户字段
        form.append("uploadFiles", this.fileList[0]);
        Axios({
          method: "post",
          url: "http://127.0.0.1:3305/querySystem/upload/uploadMaster",
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
          },
          (err) => {
            this.$message.error("上传失败");
          }
        );
      }
    },

    // 下拉框中的数据获取
    chickvalue(val) {
      var name = "";
      for (let i = 0; i <= val.length - 1; i++) {
        this.select.options.find((item) => {
          if (item.value == val[i]) {
            name += item.label;
            this.select.id = item.value + "";
          }
        });
      }
      this.addMap.customer = name;
      this.addCustomer.customer = name;
    },
    // 从服务器中获取客户名称
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

    // 上传客户
    uploadCustomer() {
      Axios({
        method: "post",
        url: "http://127.0.0.1:3305/querySystem/upload/uploadCustomer",
        data: JSON.stringify({
          customer: this.addCustomer.customer,
        }),
      }).then(
        (res) => {
          // 提示添加成功
          this.$message({
            message: "添加成功",
            type: "success",
          });
        },
        (err) => { }
      );
    },

    user() {
      this.userData.id = localStorage.getItem("ms_id");
      this.userData.username = localStorage.getItem("ms_username");
      // this.userData.role = localStorage.getItem("ms_role");
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

<style>
.body {
  width: 100%;
  height: 100%;
}

.addCustomer {
  width: 50%;
  height: 50%;
  margin-top: 12px;
  /* 居中 */
  margin-left: auto;
  margin-right: auto;
}

.input {
  width: 70%;
  height: 30px;
  border-radius: 5%;
}

.button {
  border-radius: 10%;
  margin-left: 10px;
  margin-top: 5px;
}

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