<template>
    <el-tree :data="menu" :props="defaultProps" :expand-on-click-node="false" :show-checkbox="true">
        <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="node.level<=2"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            v-if="node.childNodes.length==0"
            @click="() => open(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    
</template>

<script>
let id = 1000;

export default {
  data() {
    return {
      menu: [],
      defaultProps: {
        children: "children",
        label: "name",
      }
    };
  },

  methods: {
      //删除确认提醒框
      open(node,data) {
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.remove(node,data)
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
      },
    append(data) {
      console.log("append", data);
    },

    remove(node, data) {
      console.log("remove", node, data);
      var ids = [data.catId]
      this.$http({
        url: this.$http.adornUrl("/product/category/delete"),
        method: "post",
        data: this.$http.adornData(ids,false)
      }).then(({ data }) => {
        console.log("删除成功");
        this.getMenus();
      });
    },

    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
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