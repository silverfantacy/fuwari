---
title: Vue3 Vite 環境下解決 "runtime compilation is not supported" 問題
published: 2024-08-27
description: 在 Vue3 Vite 專案中遇到惱人的 "runtime compilation is not supported" 錯誤訊息？提供兩種有效的解決方案：善用 .vue 單檔案組件的優勢，或透過調整 Vite 配置以支援運行時編譯。
image: ''
tags: [Vue3, Vite]
category: 前端
draft: false 
---

在使用 Vue3 和 Vite 開發時，可能會遇到以下錯誤訊息：

```
[Vue warn]: Component provided template option but runtime compilation is not supported in this build of Vue. Configure your bundler to alias "vue" to "vue/dist/vue.esm-bundler.js". 
```

這個錯誤表示你在當前的 Vue 構建版本中使用了需要運行時編譯 (runtime compilation) 的功能（例如在組件中直接提供 `template` 選項），但這個構建版本並不支援運行時編譯。 Vite 預設為了最佳化效能，會預先編譯模板，因此不包含運行時編譯器。

以下是如何解決這個問題的方法：

### 1. 使用 `.vue` 單檔案組件 (Single-File Components)

這是 Vue3 推薦的做法。`.vue` 檔案允許將模板、腳本和樣式都寫在同一個檔案中，Vite 會自動處理這些檔案並進行預先編譯，避免運行時編譯的問題。

```vue
<template>
  <div>
    <h1>{{ message }}</h1>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('Hello from Vue!')
</script>
```

### 2. 如果必須在 JavaScript 中使用 `template`

如果真的需要在 JavaScript 中使用 `template` 選項，需要修改 Vite 的配置，讓它使用包含運行時編譯器的 Vue 構建版本。

**修改 `vite.config.js`**

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      vue: 'vue/dist/vue.esm-bundler.js' // 👈 這一行是關鍵
    }
  }
})
```

這個修改告訴 Vite 在解析 `import vue from 'vue'` 時，應該使用 `vue/dist/vue.esm-bundler.js` 這個檔案，這個檔案包含了運行時編譯器。

### 結論

雖然可以在 Vite 中使用運行時編譯，但建議盡量使用 `.vue` 單檔案組件，這樣可以獲得更好的效能和開發體驗。如果你遇到這個錯誤，希望這篇文章能幫助你快速解決問題。


> 參考資料：
> 
> [vue3 动态编译组件失败：Component provided template option but runtime compilation is not supported in this build of Vue](https://www.cnblogs.com/dongling/p/18091751)