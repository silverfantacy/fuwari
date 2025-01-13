---
title: 'JavaScript 滾動 API 介紹：scrollIntoView 與其他滾動方法的比較'
published: 2025-01-13
description: '介紹 scrollIntoView 的使用方式，以及與 scrollTo、scroll 等其他滾動方法的差異'
image: 'https://imager.virtualxnews.com/rest/SxJKSmK.jpeg'
tags: ['JavaScript']
category: '前端'
draft: false 
---

在網頁開發中，控制頁面滾動是一個常見的需求。之前都是用 `window.scrollTo` 來控制滾動位置，但現在有了更好用的 `scrollIntoView` 方法，可以更靈活地控制元素滾動。

## scrollIntoView 介紹

`scrollIntoView` 是一個非常實用的方法，可以讓指定的元素滾動到可視區域。

### 基本用法

```javascript
element.scrollIntoView();
// 等同於
element.scrollIntoView({ behavior: 'auto', block: 'start' });
```

### 參數選項

```javascript
element.scrollIntoView({
  behavior: 'auto' | 'smooth', // 滾動行為
  block: 'start' | 'center' | 'end' | 'nearest', // 垂直對齊
  inline: 'start' | 'center' | 'end' | 'nearest' // 水平對齊
});
```

## 其他滾動方法比較

### window.scrollTo / window.scroll

這兩個方法功能完全相同，都是用來控制視窗滾動到指定位置：

```javascript
window.scrollTo(x, y);
// 或
window.scrollTo({
  top: 100,
  left: 0,
  behavior: 'smooth'
});
```

### window.scrollBy

相對於當前位置進行滾動：

```javascript
window.scrollBy(0, 100); // 向下滾動 100px
```

## 方法比較

| 方法 | 用途 | 特點 |
|------|------|------|
| scrollIntoView | 將元素滾動到可視區域 | 最靈活，可以設定對齊方式 |
| scrollTo/scroll | 滾動到絕對位置 | 需要知道確切座標 |
| scrollBy | 相對滾動 | 適合增量滾動 |

## 實際應用範例

### 點擊按鈕滾動到指定區域

```javascript
const button = document.querySelector('.scroll-button');
const target = document.querySelector('.target-section');

button.addEventListener('click', () => {
  target.scrollIntoView({ 
    behavior: 'smooth',
    block: 'center'
  });
});
```

### 返回頂部按鈕

```javascript
function scrollToTop() {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });
}
```

## 瀏覽器支援度

- scrollIntoView：現代瀏覽器都支援，但 behavior: 'smooth' 在 Safari 15 以前不支援
- scrollTo/scroll：所有現代瀏覽器都支援
- scrollBy：所有現代瀏覽器都支援

## 結論

- `scrollIntoView` 最適合用於元素導航，特別是不知道確切座標的情況
- `scrollTo/scroll` 適合滾動到已知的確切位置
- `scrollBy` 適合需要相對滾動的場景

選擇適合的滾動方法可以讓網頁的滾動效果更加流暢自然。

> 參考資料：
> 
> [MDN - Element.scrollIntoView()](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)
>
> [MDN - Window.scrollTo()](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollTo)
