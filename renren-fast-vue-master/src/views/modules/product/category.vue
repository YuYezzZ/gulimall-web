<template>
  <div>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      :show-checkbox="true"
      draggable
      :allow-drop="allowDrop"
      node-key="catId"
      :default-expanded-keys="expandedKey"
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
          <el-button
            type="text"
            size="mini"
            @click="update(data)"
          >
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
        :model="menuForm"
        status-icon
        ref="menuForm"
        label-width="100px"
        class="demo-menuForm"
      >
        <el-form-item label="分类名称" prop="name">
          <el-input type="text" v-model="menuForm.name"></el-input>
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-input type="text" v-model="menuForm.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" prop="unit">
          <el-input type="text" v-model="menuForm.productUnit"></el-input>
        </el-form-item>
          <el-button type="primary" @click="submitCategory(menuForm)">提交</el-button>
          <el-button @click="dialogVisible=false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      menu: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      menuForm: {
        catId:"",
        name: "",
        parentCid: 0,
        children:[],
        catLevel: 1,
        sort: 0,
        icon:"",
        showStatus: 1,
        productUnit:"",
        productCount:"",
      },
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
      //分级菜单表单弹框开关
      dialogVisible: false,
      //默认展开节点
      expandedKey: [],
      //分级菜单表单弹框类型
      dialogType:"",
      dialogTitle:"",
    };
  },

  methods: {
    //判断节点是否可拖拽
    allowDrop(draggingNode, dropNode, type) {
        console.log(draggingNode, dropNode, type)
        return true;y
      },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    //提交新增分级菜单表单
    submitCategory(menuForm) {
      console.log(menuForm)
      if(this.dialogType=="add"){
        this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(menuForm, false),
      }).then(({data}) => {
        this.getMenus();
        this.expandedKey = [menuForm.parentCid];
        this.$message({
            type: "success",
            message: "添加成功!",
          });
      }).catch(() => {
          this.$message({
            type: "error",
            message: "添加失败",
          });
        });
        this.dialogVisible=false;
        this.getMenus();
        this.expandedKey = [menuForm.parentCid];
      }else{
        let {catId,name,icon,productUnit} = this.menuForm;
        this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({catId,name,icon,productUnit}, false),
      }).then(({data}) => {
        this.getMenus();
        this.expandedKey = [catId];
        this.$message({
            type: "success",
            message: "修改成功!",
          });
      }).catch(() => {
          this.$message({
            type: "error",
            message: "修改失败",
          });
        });
        this.dialogVisible=false;
        this.getMenus();
        this.expandedKey = [menuForm.catId];
      }
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
    //新增按钮
    append(data) {
      this.dialogType="add";
      this.dialogTitle="添加分类";
      this.dialogVisible = true;
      this.menuForm.parentCid=data.catId;
      this.menuForm.catLevel=data.catLevel+1;
      this.menuForm.name="";
      this.menuForm.icon="";
      this.menuForm.productUnit="";
      this.expandedKey=[data.catId]; 
      console.log("append", data);
    },
    update(data) {
      this.dialogType="update";
      this.dialogTitle="修改分类";
      this.dialogVisible = true;
      this.menuForm.name=data.name;
      this.menuForm.catId=data.catId;
      this.menuForm.icon=data.icon;
      this.menuForm.productUnit=data.productUnit;
      this.expandedKey=[data.catId]; 
      console.log("update", data);
    },
    menuTitle(){
        if(this.dialogtype=="add"){
          
        }
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
        this.expandedKey = [node.parent.data.catId];
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