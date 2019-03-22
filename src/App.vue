<template>
  <div id="app">
    <TreeGrid :columns="this.columns"
              :items="treeData"
              @on-row-click="rowClick"
              @on-sort-change="sortChange"
              @on-selection-change="selectChange" />
  </div>
</template>

<script>
import TreeGrid from "./components/TreeGrid.vue";

export default {
  name: "app",
  components: {
    TreeGrid
  },
  data() {
    const columns = [
      {
        type: "selection",
        width: 50,
        align: "center"
      },
      {
        title: "编号",
        key: "menuId",
        width: 150,
        sortable: true,
      },
      {
        title: "类型",
        key: "type",
        type: "oprationTag",
        oprations: [
          {
            val: "目录",
            type: "warning",
            term: {
              key: "type",
              val: 0
            }
          },
          {
            val: "菜单",
            type: "primary",
            term: {
              key: "type",
              val: 1
            }
          },
          {
            val: "按钮",
            type: "success",
            term: {
              key: "type",
              val: 2
            }
          }
        ]
      },
      {
        title: "操作",
        type: "oprationBtn",
        key: "opration",
        oprations: [
          {
            val: "编辑",
            type: "primary"
          },
          {
            val: "添加下级",
            type: "success"
          },
          {
            val: "删除",
            type: "info"
          }
        ]
      }
    ];
    return {
      columns,
      treeData: [{
        id: 220, // 使用 selection 必须带上 id
        menuId: 220,
        type: 0,
        children: [
          {
            id: 221,
            menuId: 221,
            type: 1
          },
          {
            id: 222,
            menuId: 222,
            type: 2,
            children: [
              {
                id: 229,
                menuId: 229,
                type: 1
              },
            ]
          },
        ]
      }, {
        id: 223,
        menuId: 223,
        type: 1,
      }]
    }
  },

  methods: {
    rowClick(result, event, index, text) {
      console.log(result, text);
    },
    sortChange(index, type) {
      console.log(index, type);
    },
    selectChange(arr) {
      console.log(arr);
    },
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
