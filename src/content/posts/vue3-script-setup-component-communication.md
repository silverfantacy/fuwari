---
title: Vue 3 <script setup>：defineExpose 父子元件之間的溝通
published: 2024-07-05
description: 'Vue 3 <script setup> 模式下，父子元件如何透過 defineExpose、provide 和 inject 進行資料傳遞與方法呼叫。'
image: 'https://media.virtualxnews.com/2024/05/f995ac3d9d57f1509658d54a5b48529c.png'
tags: [Vue 3, script setup, defineExpose, provide, inject, 父子元件, 元件溝通]
category: '前端'
draft: false 
---

## Vue 3 `<script setup>`：父子元件之間的溝通

在 Vue 3 的單檔案元件（SFC）中，`<script setup>` 是一種簡化 Composition API 使用的語法糖。它讓程式碼更簡潔，但同時也改變了父子元件之間溝通的方式。紀錄一下在 `<script setup>` 模式下，父子元件如何透過 `defineExpose`、`provide` 和 `inject` 進行資料傳遞與方法呼叫。

### 子元件向父元件傳遞資料（defineExpose）

在 `<script setup>` 中，所有定義的變數、函式預設都是私有的，無法直接被父元件取用。要讓父元件能呼叫子元件的方法或屬性，我們需要使用 `defineExpose`。

**子元件 (ChildComponent.vue)**

```vue
<script setup>
import { ref, defineExpose } from 'vue';

const childMessage = ref('訊息來自子元件');
const childMethod = () => {
  console.log('子元件的方法被呼叫了！');
};

defineExpose({
  childMessage,
  childMethod
});
</script>
```

**父元件 (ParentComponent.vue)**

```vue
<template>
  <ChildComponent ref="childRef" />
  <button @click="callChildMethod">呼叫子元件方法</button>
</template>

<script setup>
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const childRef = ref(null);

const callChildMethod = () => {
  childRef.value.childMethod(); 
  console.log(childRef.value.childMessage);
};
</script>
```

父元件透過 `ref` 取得子元件的實例，再透過 `.value` 訪問 `defineExpose` 暴露出的屬性或方法。

### 父元件向子元件傳遞資料（provide/inject）

在 `<script setup>` 中，可以使用 Vue 3 的 `provide` 和 `inject` 來實現父子元件之間的資料傳遞。

**父元件 (ParentComponent.vue)**

```vue
<script setup>
import { provide } from 'vue';

const parentMessage = '訊息來自父元件';
provide('parentMsg', parentMessage);
</script>
```

**子元件 (ChildComponent.vue)**

```vue
<script setup>
import { inject } from 'vue';

const parentMsg = inject('parentMsg');
console.log(parentMsg); 
</script>
```

父元件透過 `provide` 提供資料，子元件透過 `inject` 接收。這種方式不受元件層級限制，只要在 provide/inject 的提供與注入元件之間有祖先/後代關係即可。

### 總結

在 Vue 3 `<script setup>` 模式下，父子元件之間的溝通方式有些改變，但透過 `defineExpose`、`provide` 和 `inject`，仍然可以實現靈活的資料傳遞和方法呼叫。
