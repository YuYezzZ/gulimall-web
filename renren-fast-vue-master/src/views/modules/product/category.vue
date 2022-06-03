<template>
  <div>
    <el-switch
      v-model="draggable"
      active-color="#13ce66"
      active-text="启用拖拽"
      inactive-text="禁用拖拽"
    >
    </el-switch>
    <el-button type="danger" @click="deleteByIds()">批量删除</el-button>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      :show-checkbox="true"
      :draggable="draggable"
      :allow-drop="allowDrop"
      node-key="catId"
      :default-expanded-keys="expandedKey"
      @node-drop="handleDrop"
      getCheckedNodes
      ref="menu"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="node.level <= 2"
            @click="append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="update(data)">
            Update
          </el-button>
          <el-button
            type="text"
            size="mini"
            v-if="node.childNodes.length == 0"
            @click="open(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <!-- 修改新增表单弹框 -->
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form
        :model="category"
        :rules="rules"
        status-icon
        ref="category"
        label-width="100px"
        class="demo-category"
        hide-required-asterisk
      >
        <el-form-item label="分类名称" prop="name">
          <el-input type="text" v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-input type="text" v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" prop="unit">
          <el-input type="text" v-model="category.productUnit"></el-input>
        </el-form-item>
        <el-form-item align="right">
          <el-button type="primary" @click="submitCategory(category)"
            >提交</el-button
          >
          <el-button @click="dialogClose()">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      updateNodes: [],
      max_level: 0,
      menu: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      category: {
        catId: "",
        name: "",
        parentCid: 0,
        children: [],
        catLevel: 1,
        sort: 0,
        icon: "",
        showStatus: 1,
        productUnit: "",
        productCount: "",
      },
      rules: {
        name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 1, max: 10, message: "长度在 1 到 10个字符", trigger: "blur" },
        ],
      },
      //分级菜单表单弹框开关
      dialogVisible: false,
      //默认展开节点
      expandedKey: [],
      //分级菜单表单弹框类型
      dialogType: "",
      dialogTitle: "",
      //拖拽功能开关
      draggable: false,
    };
  },

  methods: {
    //监听拖住完成的方法
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("tree drop: ", dropNode.label, dropType, ev);
      let pCid = 0;
      let siblings = null;
      this.expandedKey = [draggingNode.data.catId];
      //当前被拖拽节点最新的父ID
      if (dropType == "inner") {
        pCid = dropNode.data.catId == undefined ? 0 : dropNode.data.catId;
        siblings = dropNode.childNodes;
      } else {
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      }
      //当前被拖拽节点的顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          if (siblings[i].level != draggingNode.level) {
            this.calDragLevelAdd1(draggingNode.data, siblings[i].level);
            this.updateNodes.push({
              catId: siblings[i].data.catId,
              sort: i,
              parentCid: pCid,
              catLevel: siblings[i].level,
            });
          } else {
            this.updateNodes.push({
              catId: siblings[i].data.catId,
              sort: i,
              parentCid: pCid,
            });
          }
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }

      //当前被拖拽节点的子节点
      console.log("updateNodes", this.updateNodes);
      this.$http({
        url: this.$http.adornUrl("/product/category/list/drop"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.getMenus();
        this.updateNodes = [];
      });
    },
    //找出节点当前所有的子节点并对level加1
    calDragLevelAdd1(node, level) {
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          this.updateNodes.push({
            catId: node.children[i].catId,
            sort: i,
            catLevel: level + 1,
          });
          this.calDragLevelAdd1(node.children[i], level + 1);
        }
      }
    },

    // 找出节点最大level
    calDragLevel(node) {
      this.max_level = node.catLevel;
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.max_level) {
            this.max_level = node.children[i].catLevel;
            this.calDragLevel(node.children[i]);
          }
        }
      }
    },
    //判断节点是否可拖拽
    allowDrop(draggingNode, dropNode, type) {
      console.log(draggingNode, dropNode, type);
      this.calDragLevel(draggingNode.data);
      let deep = this.max_level - draggingNode.data.catLevel + 1;
      console.log("deep", deep, "max_level", this.max_level);
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      }
      return deep + dropNode.parent.level <= 3;
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    //提交新增分级菜单表单
    submitCategory(category) {
      console.log(category);
      this.$refs["category"].validate((valid) => {
        if (valid) {
          if (this.dialogType == "add") {
            this.$http({
              url: this.$http.adornUrl("/product/category/save"),
              method: "post",
              data: this.$http.adornData(category, false),
            })
              .then(({ data }) => {
                this.getMenus();
                this.expandedKey = [category.parentCid];
                this.$message({
                  type: "success",
                  message: "添加成功!",
                });
              })
              .catch(() => {
                this.$message({
                  type: "error",
                  message: "添加失败",
                });
              });
            this.dialogVisible = false;
            this.getMenus();
            this.expandedKey = [category.parentCid];
          } else {
            let { catId, name, icon, productUnit } = this.category;
            this.$http({
              url: this.$http.adornUrl("/product/category/update"),
              method: "post",
              data: this.$http.adornData(
                { catId, name, icon, productUnit },
                false
              ),
            })
              .then(({ data }) => {
                this.getMenus();
                this.expandedKey = [catId];
                this.$message({
                  type: "success",
                  message: "修改成功!",
                });
              })
              .catch(() => {
                this.$message({
                  type: "error",
                  message: "修改失败",
                });
              });
            this.dialogVisible = false;
            this.getMenus();
            this.expandedKey = [category.catId];
          }
        } else {
          console.log("error submit!!");
          return false;
        }
        this.resetForm(this.category);
      });
    },
    //重置分级菜单表单
    dialogClose() {
      this.dialogVisible = false;
      this.resetForm(this.category);
    },
    resetForm() {
      console.log(this.$refs);
      this.$refs["category"].resetFields();
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
    //新增按钮
    append(data) {
      this.dialogType = "add";
      this.dialogTitle = "添加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel + 1;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.expandedKey = [data.catId];
      console.log("append", data);
    },
    update(data) {
      this.dialogType = "update";
      this.dialogTitle = "修改分类";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category = data.category;
      });
      this.dialogVisible = true;
      this.category.name = data.name;
      this.category.catId = data.catId;
      this.category.icon = data.icon;
      this.category.productUnit = data.productUnit;
      this.expandedKey = [data.catId];
      console.log("update", data);
    },
    //删除
    remove(node, data) {
      console.log("remove", node, data);
      let ids = [data.catId];
      this.$http({
        url: this.$http.adornUrl("/product/category/delete"),
        method: "post",
        data: this.$http.adornData(ids, false),
      }).then(({ data }) => {
        console.log("删除成功");
        this.getMenus();
        this.expandedKey = [node.parent.data.catId];
      });
    },
    //批量删除
    deleteByIds() {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          let data = this.$refs.menu.getCheckedNodes();
          console.log(data);
          let ids = [];
          for (const i in data) {
            if (Object.hasOwnProperty.call(data, i)) {
              ids.push(data[i].catId);
              this.expandedKey.push(data[i].parentCid);
            }
          }
          console.log(ids);
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            console.log("删除成功");
            this.getMenus();
          });
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