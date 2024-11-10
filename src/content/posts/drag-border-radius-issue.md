---
title: 解決 HTML 拖曳時圓角消失的問題
published: 2024-11-10
description: '如何解決在 Chrome 瀏覽器中拖曳圓角元素時，預覽圖示變成方形的問題。'
image: ''
tags: [Draggable, HTML, CSS, Chrome, 網頁開發]
category: '前端'
draft: false 
---

## 問題描述

在使用 HTML Drag and Drop API 進行元素拖曳時，常會遇到一個特別的問題：當拖曳具有圓角(border-radius)的元素時，拖曳時的預覽圖示會失去圓角效果，變成方形。這個問題主要出現在 Chrome 瀏覽器中，而 Firefox 和 Safari 則能正常顯示圓角效果。

## 問題原因

這個問題主要是由 Chrome 瀏覽器的渲染機制造成的。當瀏覽器生成拖曳預覽圖示時，Chrome 似乎無法正確保留元素的圓角樣式。

## 解決方案(非常的神奇)

### 方案一：使用 Transform

最簡單且有效的解決方案是在拖曳元素上添加 `transform` 屬性：

```css
.draggable-element {
  border-radius: 8px;
  transform: translate(0, 0);  /* 關鍵解決方案 */
}
```

### 方案二：設定透明度

另一個解決方案是調整元素的透明度：

```css
.draggable-element {
  border-radius: 8px;
  opacity: 0.999;  /* 設定接近但不等於 1 的透明度 */
}
```

## 實際應用範例

### 基本 HTML 結構

```html
<div 
  class="draggable-element"
  draggable="true"
>
  拖曳我試試看
</div>
```

### 完整的 CSS 樣式

```css
.draggable-element {
  padding: 16px;
  background-color: #f0f0f0;
  border-radius: 8px;
  cursor: grab;
  transform: translate(0, 0);
  
  /* 可選的其他樣式 */
  user-select: none;
  -webkit-user-drag: element;
}

/* 拖曳時的樣式 */
.draggable-element:active {
  cursor: grabbing;
}
```

### JavaScript 事件處理

```javascript
const draggableElement = document.querySelector('.draggable-element');

draggableElement.addEventListener('dragstart', (e) => {
  e.dataTransfer.setData('text/plain', 'dragged-content');
});
```

## 注意事項

1. 這個問題主要影響 Chrome 瀏覽器，其他瀏覽器可能不需要這個解決方案
2. `transform` 或 `opacity` 的設定不會影響元素的實際外觀
3. 如果使用 `opacity` 方案，建議使用 `0.999` 而不是更小的值，以避免明顯的透明效果

## 瀏覽器支援度

- Chrome：需要使用上述解決方案
- Firefox：原生支援圓角拖曳預覽
- Safari：原生支援圓角拖曳預覽
- Edge：與 Chrome 行為相同，需要解決方案

## 參考資料

- [Stack Overflow: Rounded corners with HTML draggable](https://stackoverflow.com/questions/22922761/rounded-corners-with-html-draggable)
- [MDN Web Docs: HTML Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) 
- [GitHub: react-dnd/react-dnd#788](https://github.com/react-dnd/react-dnd/issues/788)