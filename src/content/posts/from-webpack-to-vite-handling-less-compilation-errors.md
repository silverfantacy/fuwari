---
title: 從 Webpack 到 Vite 的 Less 編譯問題
published: 2024-08-21
description: '在使用 Vite 進行專案升級過程中，從 Webpack 遷移到 Vite 時，可能會遇到 Less 編譯錯誤的問題'
image: ''
tags: [Vite, Webpack, Less]
category: '前端開發'
draft: false 
---
在 Vue 專案升級過程中，從 Vue2+Webpack 遷移到 Vue3+Vite 時，可能會遇到 Less 編譯錯誤的問題。
即使已安裝 Less 相關依賴並配置 `vite.config.js`，錯誤仍可能出現。

首先，確保安裝了必要的依賴：

```bash
yarn add less less-loader
```

接著，在 `vite.config.js` 中進行如下配置：

```javascript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  css: {
    preprocessorOptions: {
      less: {
        // 在這裡添加你的 Less 配置
        javascriptEnabled: true,
      },
    },
  },
});
```

本次錯誤訊息如下：

```bash
[vite] Pre-transform error: [less] Error evaluating function `round`: argument must be a number
[vite] Internal server error: [less] Error evaluating function `round`: argument must be a number
```

這是因為 Less 在處理 `round` 函數時，傳入的參數並非數字。

為了解決此問題，可以在 `vite.config.js` 的 `css.preprocessorOptions.less.lessOptions` 中加入以下配置：

```javascript
less: {
    math: "always",
    relativeUrls: true,
    javascriptEnabled: true
},
```

透過這些配置，能順利編譯。

> 參考資料：
> 
> [How to add a loader (less-loader) to vite.js (vite.config.js)?](https://stackoverflow.com/questions/75913640/how-to-add-a-loader-less-loader-to-vite-js-vite-config-js)