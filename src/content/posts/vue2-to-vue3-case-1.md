---
title: Vue2 升級到 Vue3 - 語法差異與套件更新
published: 2024-05-05
description: '紀錄專案從 Vue2 升級到 Vue3 時，語法差異和套件更新所遇到的狀況'
image: 'https://media.virtualxnews.com/2024/05/f995ac3d9d57f1509658d54a5b48529c.png'
tags: [Vue 2, Vue 3]
category: '前端'
draft: false 
---

這篇文章將記錄升級 Vue2 到 Vue3 時的一些語法差異和套件更新。

## Vue3 與 Vue2 的語法差異

### 捨棄 this.$set

直接進行操作即可

```javascript
this.$set(element, 'isOpen', 1);
// 改用
element.isOpen = 1;
```

### 捨棄 $vnode

使用 key 或 index 替代 `$vnode`

在 `props` 中應該還需要加入 `product_item_index: {type: Number, default: 0}`。

```jsx
<div class="btn btn-danger btn-block" @click="$parent.deleteItem($vnode.key)">刪除</div>
// 改用
<div class="btn btn-danger btn-block" @click="$parent.deleteItem(product_item_index)">刪除</div>
```

### 捨棄 this.$delete

```javascript
this.$delete(items, item_index);
// 改用
items.splice(item_index, 1);
```

### 使用 v-model 替代 .sync

[vue3中使用sync实现双向数据绑定 - 掘金](https://juejin.cn/post/7148732203537530893)

```javascript
:target.sync="form.original_group_id"
// 改用
v-model:target="form.original_group_id"
```

### 棄用 ::v-deep，改用 :deep()

```scss
.a ::v-deep .b {
  /* ... */
}
// 改用
.a :deep(.b) {
  /* ... */
}
```

參考資料：
- [Vue SFC CSS Features - vuejs.org](https://vuejs.org/api/sfc-css-features.html#deep-selectors)
- [::v-deep在vue3中的替代方案是什么？ - segmentfault.com](https://segmentfault.com/q/1010000042617891)

### Vue3 中單獨包含 templete 的 HTML 無法顯示

這樣是無法顯示的：

```html
<template>
  <div>123</div>
</template>
```

要使用 template 必須要有條件判斷才能顯示：

```html
<template v-if="true">
  <div>123</div>
</template>
```

## 套件更新

### `v-clipboard` 改用 `vue-clipboard3`

::github{repo="JamieCurnow/vue-clipboard3"}

### draggable

::github{repo="SortableJS/vue.draggable.next"}

下面的 item 要改用 `#item="{ element, index }"` 的寫法，element, index 不能更改名稱。

```vue
<draggable v-model="product.carousels"
          item-key="carousels"
          class="dragArea"
          :group="{ name:'img-area', pull:'clone', put:false }">
  <template #item="{ element, index }">
    <div class="item" :key="`element-${index}`">
    </div>
  </template>
</draggable>
```

### VueMultiselect

::github{repo="shentao/vue-multiselect"}

[文件](https://vue-multiselect.js.org/)

```javascript
@input="onChanged"
// 改用
@select="onChanged"
```

> 參考資料：
>
> [Vue 3 迁移指南](https://v3-migration.vuejs.org/zh/)
>
> [Vue 2.x 至 3.0 快速升級指南](https://book.vue.tw/appendix/migration.html)