/* TreeGrid --> vue + iview
 * @Author: ceshu
 * @Date: 2018-12-14 15:12:53
 * @Last Modified by: ceshu
 * @Last Modified time: 2019-03-22 10:48:20

    Props:  - items: 树形展示数据 -columns：结构配置数据 -align: 表头偏移
    Columns: - title: 表头 -key: 渲染字段 -sortable: 排序 -width: 宽度
            - type:
              --selection: 多选框
              --oprationTag: 操作列标签  --oprationBtn：操作列按钮（参数：oprations：{val,type,term}）
    events: - @on-row-click     return(data, event, index)
            - @on-sort-change   return(arr)
            - @on-selection-change   return(key,'asc'/'desc')
 */

<template>
  <div :style="{width:tableWidth}"
    class='autoTbale'>
    <table class="table table-bordered"
      id='hl-tree-table'>
      <thead :style="{'text-align':align}">
        <tr>
          <th v-for="(column,index) in cloneColumns"
            :style="{'width':column.width+'px'}"
            :key="index">
            <label v-if="column.type === 'selection'">
              <input type="checkbox"
                v-model="checks"
                @click="handleCheckAll">
            </label>
            <label v-else>
              {{ renderHeader(column, index) }}
              <span class="ivu-table-sort"
                v-if="column.sortable">
                <Icon type="md-arrow-dropup"
                  :class="{on: column._sortType === 'asc'}"
                  @click.native="handleSort(index, 'asc')" />
                <Icon type="md-arrow-dropdown"
                  :class="{on: column._sortType === 'desc'}"
                  @click.native="handleSort(index, 'desc')" />
              </span>
            </label>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item,index) in initItems"
          :key="item.id"
          v-show="show(item)"
          :class="{'child-tr':item.parent}">
          <td v-for="(column,snum) in columns"
            :key="column.key"
            :style="tdStyle(column)">
            <label v-if="column.type === 'selection'">
              <input type="checkbox"
                :value="item.id"
                v-model="checkGroup"
                @click="handleCheckClick(item,$event,index)">
            </label>
            <div v-if="column.type === 'oprationTag'">
              <template v-for='tag in (column.oprations)'>
                <Tag :color="tag.type"
                  v-if="!tag.term || (tag.term && item[tag.term.key] === tag.term.val) "
                  size="small"
                  :key="tag.val">{{tag.val}}</Tag>
              </template>
            </div>
            <div v-if="column.type === 'oprationBtn'">
              <Button :type="opration.type"
                v-if="!opration.term || (opration.term && item[opration.term.key] === opration.term.val)"
                size="small"
                @click="rowClick(item,$event,index,opration.val)"
                v-for='opration in (column.oprations)'
                :key="opration.val">{{opration.val}}</Button>

            </div>
            <label @click="toggle(index,item)"
              v-if="!column.type">
              <span v-if='snum==iconRow()'>
                <i v-html='item.spaceHtml'></i>
                <i v-if="item.children&&item.children.length>0"
                  class="ivu-icon"
                  :class="{'ivu-icon-md-arrow-dropright':!item.expanded,'ivu-icon-md-arrow-dropdown':item.expanded }"></i>
                <i v-else
                  class="ms-tree-space"></i>
              </span> {{renderBody(item,column) }}
            </label>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
export default {
  name: "treeGrid",
  props: {
    columns: Array,
    align: {
      type: String,
      default: "center"
    },
    items: {
      type: Array,
      default: function() {
        return [];
      }
    }
  },
  data() {
    return {
      initItems: [], //处理后数据数组
      cloneColumns: [], //处理后的表头数据
      checkGroup: [], //复选框数组
      checks: false, //全选
      screenWidth: document.body.clientWidth, //自适应宽
      tdsWidth: 0, //td宽度
      timer: false, //监听时长
      dataLength: 0, //数据长度
      expand: []
    };
  },
  computed: {
    tableWidth() {
      return this.tdsWidth > this.screenWidth && this.screenWidth > 0
        ? this.screenWidth + "px"
        : "100%";
    }
  },
  watch: {
    screenWidth(val) {
      if (!this.timer) {
        this.screenWidth = val;
        this.timer = true;
        setTimeout(() => {
          this.timer = false;
        }, 400);
      }
    },
    items() {
      if (this.items) {
        this.dataLength = this.Length(this.items);
        this.initData(this.deepCopy(this.items), 1, null);
        this.checkGroup = this.renderCheck(this.items);
        if (this.checkGroup.length === this.dataLength) {
          this.checks = true;
        } else {
          this.checks = false;
        }
      }
    },
    columns: {
      handler() {
        this.cloneColumns = this.makeColumns();
      },
      deep: true
    },
    checkGroup(data) {
      this.checkAllGroupChange(data);
    }
  },
  mounted() {
    if (this.items) {
      this.dataLength = this.Length(this.items);
      this.initData(this.deepCopy(this.items), 1, null);
      this.cloneColumns = this.makeColumns();
      this.checkGroup = this.renderCheck(this.items);
      if (this.checkGroup.length === this.dataLength) {
        this.checks = true;
      } else {
        this.checks = false;
      }
    }
    // 绑定onresize事件 监听屏幕变化设置宽
    this.$nextTick(() => {
      this.screenWidth = document.body.clientWidth;
    });
    window.addEventListener("resize", () => {
      return (() => {
        window.screenWidth = document.body.clientWidth;
        this.screenWidth = window.screenWidth;
      })();
    });
  },
  methods: {
    // 有无多选框折叠位置优化
    iconRow() {
      for (var i = 0, len = this.columns.length; i < len; i++) {
        if (this.columns[i].type == "selection") {
          return 1;
        }
      }
      return 0;
    },
    // 设置td宽度,td的align
    tdStyle(column) {
      var style = {};
      if (column.align) {
        style["text-align"] = column.align;
      }
      if (column.width) {
        style["min-width"] = column.width + "px";
      }
      return style;
    },
    // 排序事件
    handleSort(index, type) {
      this.cloneColumns.forEach(col => (col._sortType = "normal"));
      if (this.cloneColumns[index]._sortType === type) {
        this.cloneColumns[index]._sortType = "normal";
      } else {
        this.cloneColumns[index]._sortType = type;
      }
      this.$emit(
        "on-sort-change",
        this.cloneColumns[index]["key"],
        this.cloneColumns[index]["_sortType"]
      );
    },
    // 点击某一行事件
    rowClick(data, event, index, text) {
      let result = this.makeData(data);
      this.$emit("on-row-click", result, event, index, text);
    },
    // 点击事件 返回数据处理
    makeData(data) {
      const t = this.type(data);
      if (t === "Array") {
        return JSON.parse(JSON.stringify(data));
      } else if (t === "Object") {
        return JSON.parse(JSON.stringify(data, replacer));
      } else {
        return data;
      }
      function replacer(key, value) {
        if (
          key != "spaceHtml" &&
          key != "parent" &&
          key != "level" &&
          key != "expanded" &&
          key != "isShow" &&
          key != "load"
        ) {
          return value;
        } else {
          return undefined;
        }
      }
    },
    // 处理表头数据
    makeColumns() {
      let columns = this.deepCopy(this.columns);
      this.tdsWidth = 0;
      columns.forEach((column, index) => {
        column._index = index;
        column._width = column.width ? column.width : "";
        column._sortType = "normal";
        this.tdsWidth += column.width ? parseFloat(column.width) : 0;
      });
      return columns;
    },
    // 数据处理 增加自定义属性监听
    initData(items, level, parent) {
      this.initItems = [];
      this.handleData(items, level, parent)
    },
    handleData(items, level, parent){
      let spaceHtml = "";
      for (var i = 1; i < level; i++) {
        spaceHtml += "<i class='ms-tree-space'></i>";
      }
      items.forEach(item => {
        item = Object.assign({}, item, {
          parent: parent,
          level: level,
          spaceHtml: spaceHtml
        });
        if (typeof item.expanded == "undefined") {
          item = Object.assign({}, item, {
            expanded: false
          });
        }
        if (typeof item.isShow == "undefined") {
          item = Object.assign({}, item, {
            isShow: false
          });
        }
        if (typeof item.isChecked == "undefined") {
          item = Object.assign({}, item, {
            isChecked: false
          });
        }
        item = Object.assign({}, item, {
          load: item.expanded ? true : false
        });
        this.initItems.push(item);
        if (item.children && item.expanded) {
          this.handleData(item.children, level + 1, item);
        }
      });
    },
    //  隐藏显示
    show(item) {
      return (
        item.level == 1 || (item.parent && item.parent.expanded && item.isShow)
      );
    },
    toggle(index, item) {
      let level = item.level + 1;
      let spaceHtml = "";
      for (var i = 1; i < level; i++) {
        spaceHtml += "<i class='ms-tree-space'></i>";
      }
      if (item.children) {
        if (item.expanded) {
          item.expanded = !item.expanded;
          this.expand.splice(this.expand.indexOf(item.id), 1);
          this.close(index, item);
        } else {
          item.expanded = !item.expanded;
          this.expand.push(item.id);
          if (item.load) {
            this.open(index, item);
          } else {
            item.load = true;
            item.children.forEach((child, childIndex) => {
              this.initItems.splice(index + childIndex + 1, 0, child);
              //设置监听属性
              this.$set(this.initItems[index + childIndex + 1], "parent", item);
              this.$set(this.initItems[index + childIndex + 1], "level", level);
              this.$set(
                this.initItems[index + childIndex + 1],
                "spaceHtml",
                spaceHtml
              );
              this.$set(this.initItems[index + childIndex + 1], "isShow", true);
              this.$set(
                this.initItems[index + childIndex + 1],
                "expanded",
                false
              );
            });
          }
        }
        this.$emit("sentExpand", this.expand); //标记默认的展开列表
      }
    },
    open(index, item) {
      if (item.children) {
        item.children.forEach((child, childIndex) => {
          child.isShow = true;
          if (child.children && child.expanded) {
            this.open(index + childIndex + 1, child);
          }
        });
      }
    },
    close(index, item) {
      if (item.children) {
        item.children.forEach((child, childIndex) => {
          child.isShow = false;
          child.expanded = false;
          if (child.children) {
            this.close(index + childIndex + 1, child);
          }
        });
      }
    },
    //点击check勾选框,判断是否有children节点 如果有就一并勾选
    handleCheckClick(data) {
      data.isChecked = !data.isChecked;
      var arr = data.children;
      if (arr) {
        if (data.isChecked) {
          this.checkGroup.push(data.id);
          for (let i = 0; i < arr.length; i++) {
            this.checkGroup.push(arr[i].id);
          }
        } else {
          for (var i = 0; i < this.checkGroup.length; i++) {
            if (this.checkGroup[i] == data.id) {
              this.checkGroup.splice(i, 1);
            }
            for (var j = 0; j < arr.length; j++) {
              if (this.checkGroup[i] == arr[j].id) {
                this.checkGroup.splice(i, 1);
              }
            }
          }
        }
      }
    },
    //checkbox 全选 选择事件
    handleCheckAll() {
      this.checks = !this.checks;
      if (this.checks) {
        this.checkGroup = this.getArray(
          this.checkGroup.concat(this.allItem(this.items))
        );
      } else {
        this.checkGroup = [];
      }
    },
    // 数组去重
    getArray(a) {
      var hash = {},
        len = a.length,
        result = [];
      for (var i = 0; i < len; i++) {
        if (!hash[a[i]]) {
          hash[a[i]] = true;
          result.push(a[i]);
        }
      }
      return result;
    },
    checkAllGroupChange(data) {
      if (this.dataLength > 0 && data.length === this.dataLength) {
        this.checks = true;
      } else {
        this.checks = false;
      }
      this.$emit("on-selection-change", this.checkGroup);
    },
    allItem(data) {
      let arr = [];
      data.forEach(item => {
        arr.push(item.id);
        if (item.children && item.children.length > 0) {
          arr = arr.concat(this.allItem(item.children));
        }
      });
      return arr;
    },
    // 返回树形数据长度
    Length(data) {
      let length = data.length;
      data.forEach(child => {
        if (child.children) {
          length += this.Length(child.children);
        }
      });
      return length;
    },
    // 返回表头
    renderHeader(column, $index) {
      if ("renderHeader" in this.columns[$index]) {
        return this.columns[$index].renderHeader(column, $index);
      } else {
        return column.title || "#";
      }
    },
    // 返回内容
    renderBody(row, column) {
      return row[column.key];
    },
    // 默认选中
    renderCheck(data) {
      let arr = [];
      data.forEach(item => {
        if (item._checked) {
          arr.push(item.id);
        }
        if (item.children && item.children.length > 0) {
          arr = arr.concat(this.renderCheck(item.children));
        }
      });
      return arr;
    },
    // 深度拷贝函数
    deepCopy(data) {
      var type = this.type(data);
      if (type === "Array" || type === "Object") {
        return JSON.parse(JSON.stringify(data));
      } else {
        return data;
      }
    },
    type(obj) {
      const match = Object.prototype.toString
        .call(obj)
        .match(/\[object (\w+)\]/);
      return match ? match[1] : "unknown";
    }
  },
  beforeDestroy() {
    window.onresize = null;
  }
};
</script>
<style scoped>
.autoTbale {
  overflow: auto;
}
table {
  width: 100%;
  border-spacing: 0;
  border-collapse: collapse;
}
.table-bordered {
  border: 1px solid #ebebeb;
}
.table > tbody > tr > td,
.table > tbody > tr > th,
.table > thead > tr > td,
.table > thead > tr > th {
  border-top: 1px solid #e7eaec;
  line-height: 1.42857;
  padding: 8px;
  vertical-align: middle;
}
.table-bordered > tbody > tr > td,
.table-bordered > tbody > tr > th,
.table-bordered > tfoot > tr > td,
.table-bordered > tfoot > tr > th,
.table-bordered > thead > tr > td,
.table-bordered > thead > tr > th {
  border: 1px solid #e7e7e7;
}
.table > thead > tr > th {
  border-bottom: 1px solid #ddd;
}
.table-bordered > thead > tr > td,
.table-bordered > thead > tr > th {
  background-color: #f5f5f6;
}
#hl-tree-table > tbody > tr {
  background-color: #fbfbfb;
}
#hl-tree-table > tbody > .child-tr {
  background-color: #fff;
}
label {
  margin: 0 8px;
}
.autoTbale >>> .ms-tree-space {
  position: relative;
  top: 1px;
  display: inline-block;
  font-style: normal;
  font-weight: 400;
  line-height: 1;
  width: 12px;
  height: 12px;
}
.ms-tree-space::before {
  content: "";
}
#hl-tree-table th > label {
  margin: 0;
}
</style>
