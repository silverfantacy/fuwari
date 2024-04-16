---
title: 瀑布流排版
published: 2024-04-09
description: '使用 Masonry 套件實現瀑布流排版'
image: 'https://media.virtualxnews.com/2024/04/8c3d88dda5a3ddd7caf61751fa08b3aa.png'
tags: ['Masonry', '瀑布流排版']
category: '前端'
draft: false 
---
使用 Masonry 套件實現瀑布流排版

在網頁開發中，我們常常需要實現瀑布流式的佈局，讓元素以不規則的方式排列，這樣可以有效地利用空間，提升用戶體驗。Masonry 套件就是一個非常方便的工具，可以幫助我們實現這樣的佈局效果。

首先，我們需要引入 Masonry 套件，可以在[https://masonry.desandro.com](https://masonry.desandro.com)下載或使用 CDN 引入。

```
npm install masonry-layout
```

接著，在我們的 HTML 文件中，我們需要建立一個容器元素，並給它一個特定的類名，例如 "msnry"：

```html
<div class="grid msnry">
  <!-- 在這裡放入要排列的元素 -->
</div>
```

在 CSS 部分，我們可以定義每個元素的樣式，例如設定元素的寬度和間距：

```css
<style scoped>
.grid-item {
  width: calc((100% - 20px) / 2);
  margin-bottom: 20px;

  @media screen and (max-width: 768px) {
    width: 100%;
  }
}
</style>
```

在 JavaScript 中，我們需要引入 Masonry 套件：

```js
import Masonry from 'masonry-layout';
```

在 Vue.js 的 `mounted` 生命週期中，我們可以初始化 Masonry：

```js
mounted() {
  this.msnry = new Masonry('.msnry', {
    itemSelector: '.grid-item',
    gutter: 20
  });
}
```

當我們新增內容到容器中時，我們需要重新啟動 Masonry，以便重新計算元素的位置：

```js
this.msnry.reloadItems();
this.msnry.layout();
```

這樣，我們就可以使用 Masonry 套件來實現網頁的瀑布流式佈局了。這個套件提供了很多自定義的選項，可以根據需求進行配置，具體的使用方法可以參考官方文檔。希望這篇文章對你有所幫助！