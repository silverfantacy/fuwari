---
title: 網頁內文中嵌入廣告的功能
published: 2024-04-21
description: InnerAd，一個在網頁內文中嵌入廣告的功能，並提供範例程式碼。
image: 'https://media.virtualxnews.com/2024/04/764657e0c3c2616a6c7cd58a175c21ea.png'
tags: [JavaScript, 網頁開發, 廣告]
category: 個人專案
draft: false 
---

個人開發的 InnerAd，在網頁中嵌入內文廣告的功能。

InnerAd 通常被稱為「飛天魔毯」的廣告樣式，它可以在使用者瀏覽文章時，切開文章顯示出來廣告的效果，並支持響應式網頁設計（RWD）。

## 使用 InnerAd

首先，在網頁想置入廣告的位置中添加 InnerAd 的 HTML 元素，這是定位用的。元素的 id 屬性應該設置為 "innerAd"，並包含你想要顯示的廣告內容（圖片）。目前限制每個頁面只能有一個 InnerAd 元素。

以下是一個範例程式碼，使用一個包含圖片的連結作為廣告內容：

```html
<div id="innerAd">
  <a href="https://laplusda.com" target="_blank" rel="noopener">
    <img src="https://fakeimg.pl/300x600/?retina=1&amp;text=300x600&amp;font=noto" />
  </a>
</div>
```

你可以根據你的需求自定義廣告內容。

接下來，你需要在你的網頁中引入 InnerAd 的 JavaScript 檔案。你可以將以下程式碼放在你的網頁中，以引入 InnerAd 的 JavaScript 檔案：

```html
<script type="text/javascript" rel="preload" as="script" src="https://scripts.laplusda.com/js/innerAd.js"></script>
```

這樣，就完成了 InnerAd 的設置。當使用者瀏覽網頁時，InnerAd 將根據程式碼的位置切開文章進行顯示。它會在瀏覽器視窗大小改變時自動調整大小和位置，以確保畫面正常。如果有任何其他問題，請隨時告訴我。

> 請注意，本文章僅用於展示 InnerAd 的功能，如果你想要實際使用 InnerAd，請聯繫我以獲取授權和相關資訊。

---

### 下面是 InnerAd 展示：

<video controls autoplay loop src="https://media.virtualxnews.com/2024/04/4e65a075899996b636ff1351434f12b5.mp4" title="InnerAd Demo" class="mx-auto" ></video>