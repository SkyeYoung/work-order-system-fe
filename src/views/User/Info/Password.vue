<template>
  <div>
    <header>
      <h2>
        修改密码
        <el-popover
          placement="bottom-start"
          width="200"
          trigger="hover"
          content="修改成功后，将会强制重新登录。"
        >
          <el-button
            class="tooltip-btn"
            slot="reference"
            icon="el-icon-question"
            type="text"
            circle
          ></el-button>
        </el-popover>
      </h2>
    </header>
    <main>
      <div class="form">
        <el-form
          :model="form"
          ref="form"
          :rules="rules"
          label-position="left"
          label-width="auto"
        >
          <el-form-item label="原密码" prop="oldPassword">
            <el-input
              v-model="form.oldPassword"
              placeholder="请输入原密码"
              show-password
              :disabled="!isDisable"
            >
              <el-button
                slot="append"
                :icon="confirmBtnIcon"
                @click="confirmOldPassword('form')"
                :loading="isConfirming"
                :disabled="!isDisable"
                >{{ confirmBtnText }}</el-button
              >
            </el-input>
          </el-form-item>

          <el-form-item label="新密码" prop="newPassword">
            <el-input
              v-model="form.newPassword"
              placeholder="请输入新密码"
              show-password
              status-icon
              :disabled="isDisable"
            ></el-input>
          </el-form-item>

          <el-form-item label="再次输入新密码" prop="newPasswordCopy">
            <el-input
              v-model="form.newPasswordCopy"
              placeholder="请确认新密码"
              show-password
              status-icon
              :disabled="isDisable"
            ></el-input>
          </el-form-item>

          <el-form-item class="btn-line">
            <el-button type="primary" @click="resetForm('form')" plain
              >重置</el-button
            >
            <el-button
              type="primary"
              @click="submitForm('form')"
              :loading="isSaving"
              :disabled="isDisable"
              >{{ submitBtnText }}</el-button
            >
          </el-form-item>
        </el-form>
      </div>
    </main>
  </div>
</template>

<script lang="ts">
import { ElForm } from "element-ui/types/form";
import Vue from "vue";

import { postData } from "../../../utils/fetchData";

export default Vue.extend({
  data() {
    const validateOldPassword = (
      rule: object,
      value: string,
      callback: (callBack: object | void) => void
    ) => {
      if (!value) {
        this.$data.isDisable = true;
        callback(new Error("原密码不能为空"));
      } else {
        if (value.length < 6 || value.length > 32) {
          callback(new Error("密码位数应大于 6 位，小于 32 位"));
        } else {
          callback();
        }
      }
    };
    const validateNewPassword = (
      rule: object,
      value: string,
      callback: (callBack: object | void) => void
    ) => {
      if (!value) {
        callback(new Error("密码不能为空"));
      } else {
        if (value === this.$data.form.oldPassword) {
          callback(new Error("输入的新密码不能与原密码相同"));
        } else {
          if (value.length < 5 || value.length > 32) {
            callback(new Error("密码位数应大于 5 位，小于 32 位"));
          } else {
            callback();
          }
        }
      }
    };
    const validateNewPasswordCopy = (
      rule: object,
      value: string,
      callback: (callBack: object | void) => void
    ) => {
      if (value !== this.$data.form.newPassword) {
        callback(new Error("输入的密码与新密码不符"));
      } else {
        callback();
      }
    };
    return {
      isDisable: true,
      isConfirming: false,
      isSaving: false,
      confirmBtn: "验证",
      // confirmBtnIcon: "",
      submitBtn: "保存修改",
      userInfo: this.$store.state.userInfo,
      form: {
        oldPassword: "",
        newPassword: "",
        newPasswordCopy: ""
      },
      rules: {
        oldPassword: [{ validator: validateOldPassword, trigger: "blur" }],
        newPassword: [{ validator: validateNewPassword, trigger: "blur" }],
        newPasswordCopy: [
          { validator: validateNewPasswordCopy, trigger: ["change", "blur"] }
        ]
      }
    };
  },
  computed: {
    confirmBtnIcon() {
      return this.$data.isConfirming ? "el-icon-check" : "";
    },
    confirmBtnText() {
      return this.$data.isConfirming ? "" : "验证";
    },
    submitBtnText() {
      return this.$data.isSaving ? "请稍后..." : "保存修改";
    }
  },
  methods: {
    confirmOldPassword(formName: string) {
      (this.$refs[formName] as ElForm).validateField(
        "oldPassword",
        (errorMsg: string) => {
          if (!errorMsg) {
            this.isConfirming = true;
            postData("/api/user/authentication", {
              worknum: this.userInfo.worknum,
              password: this.form.oldPassword
            })
              .then(() => (this.isDisable = false))
              .catch((err: string) => {
                this.isDisable = true;
                this.$message({
                  message: err || "由于未知因素，暂时无法验证密码",
                  type: "warning"
                });
              })
              .finally(() => (this.isConfirming = false));
          }
        }
      );
    },
    resetForm(formName: string) {
      (this.$refs[formName] as ElForm).resetFields();
      this.isDisable = true;
      this.isConfirming = false;
      this.isSaving = false;
    },
    submitForm(formName: string) {
      (this.$refs[formName] as ElForm).validate((valid: boolean) => {
        this.isSaving = true;
        if (valid) {
          postData("/api/user/updatePassword", {
            worknum: this.userInfo.worknum,
            password: this.form.newPassword
          })
            .then(() => {
              this.resetForm("form");
              this.$message({
                message: "修改成功，正在跳转至登录页...",
                type: "success"
              });
              setTimeout(() => {
                this.$store.commit("clearUserInfo");
                sessionStorage.clear();
                this.$router.replace({ name: "login" });
              }, 1000);
            })
            .catch((err: string) =>
              this.$message({
                message: err || "出现未知错误，暂时无法修改密码",
                type: "warning"
              })
            );
        }
      });
    }
  }
});
</script>

<style lang="scss" scoped>
header,
main {
  margin-left: 15px;
}

.tooltip-btn {
  padding: 0%;
}

.form {
  width: 30%;
}

.btn-line {
  display: flex;
  justify-content: flex-end;
}
</style>
