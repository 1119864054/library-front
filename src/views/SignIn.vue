<template>
  <el-container :style="main_style">
    <vue-particles
      id="particles-js"
      color="#4b9fe9"
      :particleOpacity="0.7"
      :particlesNumber="60"
      shapeType="circle"
      :particleSize="4"
      linesColor="#b6d9f5"
      :linesWidth="1"
      :lineLinked="true"
      :lineOpacity="0.4"
      :linesDistance="150"
      :moveSpeed="3"
      :hoverEffect="true"
      hoverMode="grab"
      :clickEffect="false"
      clickMode="false"
    >
    </vue-particles>
    <el-main>
      <el-row>
        <el-col
          :xs="24"
          :sm="{ span: 18, offset: 6 }"
          :md="{ span: 14, offset: 10 }"
          :lg="{ span: 8, offset: 14 }"
          :xl="{ span: 6, offset: 16 }"
        >
          <el-form
            :model="ruleForm"
            status-icon
            :rules="rules"
            ref="ruleForm"
            label-width="100px"
            style="margin-top: 60px; margin-right: 30px"
          >
            <el-form-item>
              <h1>用户登录</h1>
            </el-form-item>
            <el-form-item label="用户名/邮箱" prop="userInfo">
              <el-input
                v-model="ruleForm.userInfo"
                clearable
                autocomplete="on"
                placeholder="输入用户名或邮件地址"
              ></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input
                v-model="ruleForm.password"
                show-password
                clearable
                placeholder="输入密码"
                @keyup.enter="submitForm('ruleForm')"
              ></el-input>
            </el-form-item>
            <el-alert
              v-if="code !== ''"
              :title="msg"
              type="info"
              :closable="false"
              show-icon
            >
            </el-alert>
            <el-form-item>
              <span>没有账号？</span>
              <el-button type="text" @click="to_signUp">注册</el-button>
            </el-form-item>
            <el-form-item>
              <el-button
                type="primary"
                @click="submitForm('ruleForm')"
                :loading="signIn_ing"
                >{{ signIn_ing ? "登录中..." : "登录" }}</el-button
              >
              <el-button @click="resetForm('ruleForm')">重置</el-button>
            </el-form-item>
          </el-form>
        </el-col></el-row
      >
    </el-main>
  </el-container>
</template>

<script>
import { request } from "@/network/request";
import main_bg from "../assets/img/bg2.jpg";
import store from "@/store";
import { ElMessage } from "element-plus";

export default {
  data() {
    let checkUserInfo = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("用户名不能为空"));
      } else {
        callback();
      }
    };
    let validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        callback();
      }
    };
    return {
      msg: "",
      code: "",
      signIn_ing: false,
      main_style:
        "height:100%;background: url(" +
        main_bg +
        ") no-repeat center; background-size: cover; border-radius: 10px;",
      ruleForm: {
        userInfo: "",
        password: ""
      },
      rules: {
        userInfo: [{ validator: checkUserInfo, trigger: "blur" }],
        password: [{ validator: validatePass, trigger: "blur" }]
      }
    };
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.sign_in();
        } else {
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    sign_in: function() {
      console.log("开始登录...");
      this.signIn_ing = true;
      request({
        url: "/user/login",
        method: "post",
        data: {
          userInfo: this.ruleForm.userInfo,
          password: this.ruleForm.password
        }
      })
        .then(res => {
          // console.log(res);
          this.code = res.data.code;
          this.msg = res.data.msg;
          let userInfo = JSON.parse(res.config.data);
          userInfo.userId = res.data.result;
          if (res.data.code === "200") {
            store.commit("setUserInfo", userInfo);
            ElMessage.success("登录成功，正在跳转到主页...");
            setTimeout(this.to_home, 800);
          } else if (res.data.code === "400") {
            ElMessage.error(res.data.msg);
          }
        })
        .catch(err => {
          console.log(err);
          ElMessage.error("请求超时！");
        })
        .finally(() => {
          this.signIn_ing = false;
        });
    },
    to_signUp: function() {
      this.$router.push({ name: "SignUp" });
    },
    to_home: function() {
      this.$router.push({ name: "Home" });
    }
  },
  beforeCreate() {
    let user = store.getters.userInfo;
    if (user !== null && user !== undefined) {
      this.$router.replace({ name: "Home" });
    }
  }
};
</script>

<style lang="css">
#particles-js {
  width: 100%;
  height: calc(100% - 100px);
  position: absolute;
}
</style>
