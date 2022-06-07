<template>
  <el-cascader
    placeholder="试试搜索：指南"
    :options="options"
    filterable
    :props="defaultProps"
    ref="options"
  ></el-cascader>
</template>

<script>
export default {
  data() {
    return {
      options: [],
      defaultProps: {
        value: "catId",
        children: "children",
        label: "name",

      },
    };
  },

  methods: {
    getOptions() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.options = data.data;
        console.log("级联数据", this.options);
      });
    },
  },
  created() {
    this.getOptions();
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