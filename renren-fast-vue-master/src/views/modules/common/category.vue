<template>
    <el-tree
      :data="menu"
      :props="defaultProps"
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span  slot-scope="{ node}">
        <span>{{ node.label }}</span>
      </span>
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
      //默认展开节点
      expandedKey: [],
    };
  },

  methods: {
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