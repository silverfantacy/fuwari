---
title: 使用 JavaScript 計算螢幕刷新率
published: 2024-10-18
description: '使用 JavaScript 來獲取當前螢幕的刷新率。'
image: ''
tags: [JavaScript, 螢幕刷新率, requestAnimationFrame]
category: '前端'
draft: false 
---

## 使用 JavaScript 計算螢幕刷新率

在開發網頁動畫或遊戲時，了解螢幕的刷新率對於優化性能和提供流暢的用戶體驗至關重要。將介紹如何使用 JavaScript 來獲取當前螢幕的刷新率。

### 原理與方法

要使用 JavaScript 來獲取當前螢幕的刷新率，我們可以通過監聽螢幕的刷新事件並計算時間間隔來實現。主要方法是使用 `requestAnimationFrame` 函數。

### 使用 `requestAnimationFrame`

`requestAnimationFrame` 是一個用於創建動畫的函數，它會在瀏覽器準備重繪之前調用指定的回調函數。通過記錄每次調用的時間戳，我們可以計算出螢幕的刷新率。

### 範例代碼

以下是一個使用 JavaScript 計算螢幕刷新率的範例：

```javascript
let lastTime = performance.now();
let frameCount = 0;
let refreshRate = 0;

function calculateRefreshRate() {
    const currentTime = performance.now();
    frameCount++;

    // 每秒計算一次刷新率
    if (currentTime - lastTime >= 1000) {
        refreshRate = frameCount; // 每秒的幀數即為刷新率
        console.log('螢幕刷新率：' + refreshRate + ' Hz');
        frameCount = 0; // 重置計數
        lastTime = currentTime; // 更新最後時間
    }

    requestAnimationFrame(calculateRefreshRate);
}

// 啟動計算
requestAnimationFrame(calculateRefreshRate);
```

### 代碼解釋

1. 使用 `performance.now()` 來獲取高精度的時間戳。
2. `frameCount` 用於記錄每秒的幀數。
3. 在 `calculateRefreshRate` 函數中，計算時間差，當達到或超過 1 秒時，計算並輸出刷新率。
4. 使用 `requestAnimationFrame` 遞迴調用函數，確保持續監測。