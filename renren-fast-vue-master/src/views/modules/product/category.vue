<template>
  <el-tree
    :data="menu"
    :props="defaultProps"
    :expand-on-click-node="false"
    :show-checkbox="true"
  >
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button
          type="text"
          size="mini"
          v-if="node.level <= 2"
          @click="() => append(data)"
        >
          Append
        </el-button>
        <el-button
          type="text"
          size="mini"
          v-if="node.childNodes.length == 0"
          @click="() => open(node, data)"
        >
          Delete
        </el-button>
      </span>
      <!-- 修改新增表单弹框 -->
      <el-dialog
        title="提示"
        :visible.sync="dialogVisible"
        width="30%"
        :before-close="handleClose"
      >
        <el-form
          :model="ruleForm"
          status-icon
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="密码" prop="pass">
            <el-input
              type="password"
              v-model="ruleForm.pass"
              autocomplete="off"
            ></el-input>
          </el-form-item>
          <el-form-item label="确认密码" prop="checkPass">
            <el-input
              type="password"
              v-model="ruleForm.checkPass"
              autocomplete="off"
            >
            </el-input>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model.number="ruleForm.age"></el-input>
          </el-form-item>
          <el-form-item label="菜单等级" prop="level">
            <el-select v-model="select"  placeholder="请选择">
              <el-option label="一级菜单" value="1"></el-option>
              <el-option label="二级菜单" value="2"></el-option>
              <el-option label="三级菜单" value="3"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')"
              >提交</el-button
            >
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
          
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </span>
  </el-tree>
</template>

<script>
let id = 1000;

export default {
  data() {
    var checkAge = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("年龄不能为空"));
      }
      setTimeout(() => {
        if (!Number.isInteger(value)) {
          callback(new Error("请输入数字值"));
        } else {
          if (value < 18) {
            callback(new Error("必须年满18岁"));
          } else {
            callback();
          }
        }
      }, 1000);
    };
    var validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        if (this.ruleForm.checkPass !== "") {
          this.$refs.ruleForm.validateField("checkPass");
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleForm.pass) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      menu: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      ruleForm: {
        pass: "",
        checkPass: "",
        age: "",
      },
      rules: {
        pass: [{ validator: validatePass, trigger: "blur" }],
        checkPass: [{ validator: validatePass2, trigger: "blur" }],
        age: [{ validator: checkAge, trigger: "blur" }],
      },
      // menuForm: {
      //   catId: "",
      //   name: "",
      //   parentCid: "",
      //   catLevel: "",
      //   sort: "",
      //   icon: "",
      //   productUnit,
      //   productCount: "",
      // },
      // rules: {
      //   name: [{ validator: nut, trigger: "blur" }],
      //   parentCid: [{ validator: validatePass2, trigger: "blur" }],
      //   age: [{ validator: checkAge, trigger: "blur" }],
      //   catLevel: [{ validator: checkAge, trigger: "blur" }],
      //   sort: [{ validator: checkAge, trigger: "blur" }],
      //   icon: [{ validator: checkAge, trigger: "blur" }],
      //   productUnit: [{ validator: checkAge, trigger: "blur" }],
      //   productCount: [{ validator: checkAge, trigger: "blur" }],
      // },
      //修改分级菜单表单弹框开关
      dialogVisible: false,
    };
  },

  methods: {
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    //新增分级菜单表单
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert("submit!");
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    //重置分级菜单表单
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    //删除确认提醒框
    open(node, data) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.remove(node, data);
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    append(data) {
      this.dialogVisible = true;
      // console.log("append", data);
    },

    remove(node, data) {
      console.log("remove", node, data);
      var ids = [data.catId];
      this.$http({
        url: this.$http.adornUrl("/product/category/delete"),
        method: "post",
        data: this.$http.adornData(ids, false),
      }).then(({ data }) => {
        console.log("删除成功");
        this.getMenus();
      });
    },

    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menu = data.data;
        console.log("数据", this.menu);
      });
    },
  },
  activated() {
    this.getMenus();
  },
};
</script>
<style  scoped>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>