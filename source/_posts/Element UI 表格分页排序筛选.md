---
title: Element UI 表格分页排序筛选
date: 2020-06-12 11:50:15
tags: 
- vue
- element-ui
categories: vue
---
Element UI表格有自带的排序和筛选功能，但是只能排序和筛选当前页。

如果涉及到分页，官方的推荐做法是后台排序筛选，但是如果后台真的一次返回所有数据，那就需要前台分页筛选了。

官网也提供了自定义筛选和自定义排序的方法。<https://element.eleme.cn/#/zh-CN/component/table>

下面介绍分页排序和筛选的方法

## 1. 数据

```js
table: {
    data: [],           // 表格原始数据
    pageData: [],       // 分页后的数据
    sort: {},           // 排序的内容，只会有一个或没有
    filters: {},        // 过滤的内容，可以有多个
    currentPage: 1,     // 当前页
    pageSize: 10,       // 每页的条数
    total: 0            // 过滤和排序后的数据的总量
}
```

## 2. 方法

### 2.1. vue 中的 methods

```js
// 获取当前列的筛选内容
getFilters(name) {
    return this.$table.getFilters(this.table.data, name);
},
// 处理筛选条件变化
handleFilterChange(filter) {
    let key = Object.keys(filter)[0];
    this.table.filters[key] = filter[key];
    this.table.currentPage = 1;
    this.$table.handleTableChange(this.table);
},
// 处理排序条件变化
handleSortChange({ column, prop, order }) {
    this.table.sort = { prop: prop, order: order };
    this.$table.handleTableChange(this.table);
},
// 处理分页条数变化
handleSizeChange(val) {
    this.table.pageSize = val;
    this.$table.handleTableChange(this.table);
},
// 处理当前页变化
handleCurrentChange(val) {
    this.table.currentPage = val;
    this.$table.handleTableChange(this.table);
},
```

### 2.2. table.js

封装了表格相关的函数

```js
export function getFilters(data, name) {
    if (data == null) {
        return;
    }
    let list = [];
    data.forEach(element => {
        list.push(element[name]);
    });
    list = [...new Set(list)];
    let filters = [];
    list.forEach(element => {
        filters.push({ text: element, value: element });
    });
    return filters;
};

export function handleTableChange(table) {
    let data = [];
    table.data.forEach(element => {
        let flag = true;
        for (let key in table.filters) {
            if (table.filters[key].length === 0) {
                continue;
            }
            let set = new Set(table.filters[key]);
            if (!set.has(element[key])) {
                flag = false;
                break;
            }
        }
        if (flag) {
            data.push(element);
        }
    });
    if (table.sort.order != null) {
        data.sort(function(a, b) {
            if (table.sort.order == "ascending") {
                if (a[table.sort.prop] > b[table.sort.prop]) {
                    return 1;
                }
            } else if (table.sort.order == "descending") {
                if (a[table.sort.prop] < b[table.sort.prop]) {
                    return 1;
                }
            }
            return -1;
        })
    }
    table.total = data.length;
    table.pageData = data.slice((table.currentPage - 1) * table.pageSize, table.currentPage * table.pageSize);
};
```

## 3. 表格属性

### 3.1. el-table

```html
<el-table
    :data="pageData"
    border
    @filter-change="handleFilterChange"
    @sort-change="handleSortChange"
    :header-cell-style="headerCellStyle"
>
```

### 3.2. el-table-column

```html
<el-table-column
    prop="time"
    label="时间"
    column-key="time"
    :filters="getFilters('time')"
    filter-placement="bottom-start"
    sortable="custom"
></el-table-column>
```
