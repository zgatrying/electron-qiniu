<template>
  <section
    style="-webkit-app-region: drag"
    class="login"
  >
    <div class="login-form">
      <h2 class="title"><img
          :src="logo"
          alt="logo"
        ></h2>
      <el-form
        :model="mac"
        ref="mac"
      >
        <el-form-item
          prop="accessKey"
          :rules="{ required: true, message: 'AccessKey 为空或者错误', trigger: 'blur' }"
        >
          <el-input
            v-model="mac.accessKey"
            placeholder="请输入 AccessKey"
            type="password"
          ></el-input>
        </el-form-item>
        <el-form-item
          prop="secretKey"
          :rules="{ required: true, message: 'SecretKey 为空或者错误', trigger: 'blur' }"
        >
          <el-input
            v-model="mac.secretKey"
            placeholder="请输入 SecretKey"
            type="password"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
            class="btn"
            type="primary"
            @click="submitForm('mac')"
            :disabled="submitState"
          >登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </section>
</template>

<script>
import { getBucketList } from "@/service/getData.js";
import { logo } from "@/assets/image.js";
import { getToken } from "@/utils/common";

export default {
  data() {
    return {
      mac: {
        accessKey: "",
        secretKey: ""
      },
      submitState: false,
      logo: ""
    };
  },
  mounted() {
    this.logo = logo;
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.getBucket();
        } else {
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    getBucket() {
      this.submitState = true;
      getBucketList(this.mac)
        .then(it => {
          this.submitState = false;
          if (it.data.length) {
            this.$electron.ipcRenderer.send("bucketsList", {
              accessKey: this.mac.accessKey,
              secretKey: this.mac.secretKey
            });
            this.$store.dispatch("SetToken", this.mac);
            this.$electron.ipcRenderer.send("switchToHome");
          }
        })
        .catch(e => {
          this.submitState = false;
          // 异步验证表单
          this.resetForm("mac");
          this.submitForm("mac");
        });
    },
    checkLogin() {
      let login = getToken();
      if (JSON.stringify(login) !== "{}") {
        this.$electron.ipcRenderer.send("switchToHome");
      }
    }
  },
  created() {
    this.checkLogin();
  }
};
</script>

<style rel="stylesheet/scss" lang="scss">
.login {
  background: #1989fa;
  width: 100%;
  height: 100%;
  .login-form {
    width: 260px;
    margin: 0 auto;
    background: #fff;
    top: 50%;
    left: 50%;
    position: absolute;
    transform: translate(-50%, -50%);
    box-shadow: 0 1px 5px 0px rgba(0, 0, 0, 0.1);
    border-radius: 4px;
    padding: 25px;
    .title {
      text-align: center;
      font-size: 18px;
      line-height: 26px;
      margin-bottom: 35px;
      img {
        width: 80px;
      }
    }
    .form-group {
      margin-bottom: 15px;
    }
    .form-control {
      display: block;
      width: 100%;
      height: 34px;
      padding: 6px 12px;
      font-size: 14px;
      line-height: 1.42857143;
      color: #555;
      background-color: #fff;
      background-image: none;
      border: 2px solid #e1e6f0;
      border-radius: 4px;
      -webkit-transition: border-color ease-in-out 0.15s,
        -webkit-box-shadow ease-in-out 0.15s;
      -o-transition: border-color ease-in-out 0.15s,
        box-shadow ease-in-out 0.15s;
      transition: border-color ease-in-out 0.15s, box-shadow ease-in-out 0.15s;
    }
  }
  .btn {
    width: 100%;
    /* background: #1989fa; */
  }
  .btn {
    width: 100%;
    /* background: #1989fa; */
  }
  ::-webkit-input-placeholder {
    /* WebKit browsers */
    color: #e1e6f0;
  }
  :-moz-placeholder {
    /* Mozilla Firefox 4 to 18 */
    color: #e1e6f0;
  }
  ::-moz-placeholder {
    /* Mozilla Firefox 19+ */
    color: #e1e6f0;
  }
  :-ms-input-placeholder {
    /* Internet Explorer 10+ */
    color: #e1e6f0;
  }
}
</style>