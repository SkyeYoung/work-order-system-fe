<template>
  <div>
    <div>
      <el-form
        :class="{ 'is-disable': isDisable }"
        ref="form"
        :model="form"
        size="medium"
        label-position="right"
        label-width="160px"
      >
        <el-form-item class="form-item" label="项目名称" prop="project">
          <el-input
            v-model="form.project"
            placeholder="请输入项目名称"
          ></el-input>
        </el-form-item>

        <el-form-item class="form-item" label="负责人" prop="master">
          <el-input v-model="form.master" placeholder="请输入负责人"></el-input>
        </el-form-item>

        <el-form-item class="form-item" label="类别" prop="type">
          <el-input v-model="form.type" placeholder="请输入类别"></el-input>
        </el-form-item>

        <el-form-item class="form-item" label="院部" prop="department">
          <el-select
            v-model="form.department"
            placeholder="请选择，或输入以查找"
            filterable
          >
            <el-option
              v-for="item in options.department"
              :key="item.id"
              :label="item.dptName"
              :value="item.dptName"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="奖金计算科室" prop="computeOffice">
          <el-select v-model="form.computeOffice" placeholder="请选择">
            <el-option
              v-for="item in computeOfficeList"
              :key="item"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item class="form-item" label="立项年度" prop="year">
          <el-date-picker
            align="center"
            v-model="form.year"
            type="year"
            value-format="yyyy"
            placeholder="请选择立项年度"
          ></el-date-picker>
        </el-form-item>

        <el-form-item class="form-item" label="奖金" prop="bonus">
          <el-input
            v-model.number="form.bonus"
            placeholder="请输入奖金"
          ></el-input>
        </el-form-item>
      </el-form>
    </div>

    <div class="btn-line">
      <el-button @click="reset('form')" type="primary" plain>重置</el-button>
      <el-button :loading="isDisable" @click="addInfo" type="primary">{{
        saveBtnText
      }}</el-button>
    </div>
  </div>
</template>

<script lang="ts">
import { ElForm } from "element-ui/types/form";
import Vue from "vue";

import { Department } from "@/interface/list-data";
import { computeOffice } from "@/static-data/work-order";
import { fetchDepartmentList, postData } from "@/utils/fetchData";

interface Data {
  id: number;
  department: string;
  computeOffice: string;
  type: string;
  year: string;
  project: string;
  master: string;
  bonus: number;
  status: number | string;
  lastTime: string;
}

export default Vue.extend({
  data() {
    return {
      token: this.$store.state.userInfo.token,
      computeOfficeList: computeOffice,
      isDisable: false,
      form: {
        id: 0,
        department: "",
        computeOffice: "",
        type: "",
        year: "",
        project: "",
        master: "",
        bonus: 0,
        status: "",
        lastTime: ""
      },
      options: {
        department: []
      }
    };
  },
  methods: {
    reset(formName: string) {
      (this.$refs[formName] as ElForm).resetFields();
    },
    addInfo() {
      this.isDisable = true;
      postData("/api/root/bonus/add", this.form)
        .then(() =>
          this.$message({
            message: "奖金信息添加成功",
            type: "success"
          })
        )
        .catch((err: string) => {
          this.isDisable = false;
          this.$message({
            message: err || "未知错误",
            type: "warning"
          });
        })
        .finally(() => (this.isDisable = false));
    }
  },
  created() {
    // 请求院部列表
    fetchDepartmentList()
      .then(
        (data: Department[]) =>
          ((this.options.department as Department[]) = data)
      )
      .catch((err: string) => {
        this.$message({
          message: err || "由于未知因素，无法获取院部列表",
          type: "warning"
        });
      });
  },
  computed: {
    saveBtnText() {
      return this.$data.isDisable ? "正在保存..." : "添加";
    }
  }
});
</script>

<style lang="scss" scoped>
.form-item {
  width: 450px;
}

.el-form {
  height: 78vh;
  width: inherit;
  overflow: auto;
}

.btn-line {
  position: absolute;
  bottom: 20px;
  right: 50px;
}

.is-disable {
  filter: blur(10px);
  cursor: not-allowed;
}
</style>
