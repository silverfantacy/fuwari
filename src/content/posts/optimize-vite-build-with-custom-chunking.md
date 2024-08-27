---
title: Vite 性能優化：客製化拆包策略，提升應用程式加載速度
published: 2024-08-27
description: 在 Vite 專案中，透過 vite.config.js 的配置，實現客製化的程式碼拆包。
image: ''
tags: [Vite, 打包, 性能優化]
category: 前端
draft: false 
---

在現代 Web 開發中，應用程式往往依賴大量的第三方套件。這些套件雖然功能強大，但也會增加應用程式的體積，導致加載速度變慢。Vite 作為一款新興的前端構建工具，提供了靈活的配置選項，讓我們可以透過打包優化來提升應用程式的性能。

將介紹如何在 `vite.config.js` 中配置客製化的程式碼拆包邏輯，將第三方套件分類打包，從而減少請求數量，提升應用程式的加載速度。

## 客製化拆包策略

首先需要在 `vite.config.js` 的 `build.rollupOptions.output` 中配置拆包邏輯。以下是一個範例配置：

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { dependencies } from './package.json'; // 引入 dependencies

export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            const dependenciesKeys = Object.keys(dependencies);
            const match = dependenciesKeys.find((item) => id.includes(item));

            // 不拆分的套件
            const noSplit = ['some-package']; // 這些套件不會被拆分

            // 獨立打包的套件
            const independent = ['some-package']; // 這些套件會被單獨打包

            if (match) {
              if (noSplit.includes(match)) {
                return; // 不拆分，返回 undefined
              } else if (independent.includes(match)) {
                return match; // 單獨打包特定套件
              } else {
                return 'vendor'; // 其他套件打包到 vendor，可以自行定義名稱
              }
            }
          }
        },
      },
    },
  },
});
```

在這個配置中，我們首先從 `package.json` 中引入 `dependencies`，以便獲取專案中所有第三方套件的名稱。然後，在 `manualChunks` 函數中，我們對每個模組的 ID 進行檢查：

1. **是否屬於第三方套件**： 檢查模組 ID 是否包含 `node_modules`。
2. **是否需要特殊處理**： 
    * 如果在 `noSplit` 列表中，則不拆分，直接返回 `undefined`。
    * 如果在 `independent` 列表中，則單獨打包成一個 chunk，chunk 名稱即為套件名。
    * 其他情況下，打包到名為 'vendor' 的 chunk 中。`vendor` 是一個通用的名稱，可以自行定義。

## 優點與注意事項

透過客製化的拆包策略，我們可以獲得以下優點：

* **減少請求數量**：將多個套件打包成一個檔案，可以減少瀏覽器發送的請求數量，從而提升加載速度。
* **利用瀏覽器緩存**：打包後的檔案通常變化較少，瀏覽器可以將其緩存起來，加快後續訪問的速度。
* **靈活管理特定套件**：可以根據需要，將特定套件單獨打包或不拆分，實現更靈活的管理。
* **客製化拆包策略**：您可以根據專案的實際情況，自由調整拆包的邏輯，實現更細緻的優化。

然而，也需要注意以下事項：

* **初始加載時間**：打包後的檔案可能會比較大，導致初始加載時間稍有增加。
* **拆包策略的複雜性**：客製化拆包策略需要謹慎設計，避免過度拆分或不合理的打包，反而影響性能。


> 參考資料：
> 
> [使用vue3 vite构建项目，优化性能，拆分包，减小主包js的大小，优化打包内存占用和速度](https://www.jianshu.com/p/54117a65264e)