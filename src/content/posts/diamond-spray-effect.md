---
title: 使用 HTML、CSS 和 JavaScript 實現鑽石噴灑效果
published: 2024-10-20
description: '使用前端技術實現互動式鑽石噴灑動畫效果'
image: ''
tags: [HTML, CSS, JavaScript, GSAP, 動畫效果]
category: '前端'
draft: false 
---

## 鑽石噴灑效果實現

主要是使用 GSAP 來實現動畫效果，並且管理 DOM 元素，減少 DOM 操作的頻率。

### 基礎架構

1. HTML 結構
```html
<!DOCTYPE html>
<html>
<head>
  <title>鑽石噴灑效果</title>
</head>
<body>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.5/gsap.min.js"></script>
</body>
</html>
```

2. CSS 樣式
```css
.diamond {
  width: 60px;
  height: 60px;
  position: absolute;
  will-change: transform, opacity;
  background-image: url('diamond.png');
  background-size: cover;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.2));
}
```

### 核心實現

1. 管理目標物件
```javascript
const DIAMOND_COUNT = 10;
const DIAMOND_POOL_SIZE = 30;

function createDiamondPool() {
  for (let i = 0; i < DIAMOND_POOL_SIZE; i++) {
    const diamond = document.createElement("div");
    diamond.classList.add("diamond");
    diamond.style.display = 'none';
    document.body.appendChild(diamond);
    diamondPool.push(diamond);
  }
}
```

2. 動畫控制
```javascript
function animateDiamond(diamond, clickX, clickY) {
  const [endX, endY] = getRandomPointInCircle(clickX, clickY, 100);
  
  // 噴射動畫
  gsap.to(diamond, {
    duration: 0.4,
    x: endX - 30,
    y: endY - 30,
    scale: 0.8 + Math.random() * 0.4,
    rotation: (Math.random() - 0.5) * 60,
    ease: "power2.out",
    onComplete: handleFallAnimation
  });
}

function handleFallAnimation(diamond) {
  gsap.to(diamond, {
    duration: 0.6,
    y: window.innerHeight + 50,
    opacity: 0,
    ease: "power1.in",
    onComplete: () => returnDiamond(diamond)
  });
}
```

### 效能優化

1. 使用對象池減少 DOM 操作
2. 應用 `will-change` 優化動畫性能
3. 實現點擊節流控制
```javascript
const CLICK_THRESHOLD = 100;
let lastClickTime = 0;

document.addEventListener("click", (event) => {
  if (Date.now() - lastClickTime < CLICK_THRESHOLD) return;
  lastClickTime = Date.now();
  
  createDiamonds(event.clientX, event.clientY);
});
```

### 範例

<p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="bGXrmmr" data-pen-title="鑽石噴灑效果" data-preview="true" data-user="Serment" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/Serment/pen/bGXrmmr">
  鑽石噴灑效果</a> by Zero (<a href="https://codepen.io/Serment">@Serment</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>