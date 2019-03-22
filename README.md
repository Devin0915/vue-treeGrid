# vue-treeGrid

```
vue + iview
```

## Project setup

```
yarn install
```

### Compiles and hot-reloads for development

```
yarn run serve
```

## Config

### Props

| Props   | Explain      |
| ------- | ------------ |
| items   | 树形展示数据 |
| columns | 结构配置数据 |
| align   | 表头偏移     |

### Columns

| Columns  | Explain                          |
| -------- | -------------------------------- |
| title    | 表头                             |
| key      | 渲染字段                         |
| sortable | 排序                             |
| width    | 宽度                             |
| type     | selection 多选框 / opration 操作 |

* opration: 
  - oprationTag：操作列标签; 
  - oprationBtn：操作列按钮（参数：oprations：{val,type,term}）
* 使用 selection需要传入id

### Events

| Events               | Return                             |
| -------------------- | ---------------------------------- |
| @on-row-click        | return (result, event, index, text) |
| @on-sort-change      | return (arr)                        |
| @on-selection-change | return (key,'asc'/'desc')           |

## Demo
![demo.png](src/assets/demo.png)