<template>
    <el-tree
      :data="menu"
      :props="defaultProps"
      node-key="catId"
      @node-click="nodeclick"
      ref="menu"
    >
    </el-tree>
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
    };
  },

  methods: {
    // 节点点击事件传给父类
    nodeclick(data,node,component){
      console.log("111",data,node,component);
      this.$emit("tree-node-click",data,node,component)
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