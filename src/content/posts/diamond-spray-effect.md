---
title: 使用 HTML、CSS 和 JavaScript 實現鑽石噴灑效果
published: 2024-10-20
description: '詳細解析如何使用 HTML、CSS 和 JavaScript 創建一個互動式的鑽石噴灑效果，包括對象池優化和 GSAP 動畫庫的應用。'
image: ''
tags: [HTML, CSS, JavaScript, GSAP, 動畫效果]
category: '前端'
draft: false 
---

## 鑽石噴灑效果實現

### HTML 結構

首先，我們需要建立基本的 HTML 結構：

```html
<!DOCTYPE html>
<html>
<head>
  <title>鑽石噴灑效果</title>
  <style>
    /* CSS 樣式將在這裡 */
  </style>
</head>
<body>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.5/gsap.min.js"></script>
  <script>
    // JavaScript 代碼將在這裡
  </script>
</body>
</html>
```

這個結構包含了頁面的基本元素，並引入了 GSAP（GreenSock Animation Platform）庫，我們將使用它來創建流暢的動畫效果。

### CSS 樣式

接下來，我們定義鑽石元素的樣式：

```css
body {
  overflow: hidden;
  background-color: #f0f0f0;
  perspective: 1000px;
}
.diamond {
  width: 60px;
  height: 60px;
  position: absolute;
  will-change: transform, opacity;
  background-image: url('https://coins-game.zeabur.app/images/%E6%8E%89%E8%90%BD%E7%89%A9/%E9%91%BD%E7%9F%B3/Animation_Diamandp_@4x.png');
  background-size: cover;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.2));
}
```

這些樣式設定了鑽石的大小、位置和外觀。`will-change` 屬性用於優化動畫性能，而 `filter` 則添加了陰影效果。

### JavaScript 實現

現在，讓我們逐步實現鑽石噴灑的效果：

1. 定義常量和變量：

```javascript
const DIAMOND_COUNT = 10; // 每次產生的鑽石數量
const DIAMOND_POOL_SIZE = 30; // 對象池大小
const diamondPool = [];
let activeDiamonds = 0;
```

2. 創建鑽石對象池：

```javascript
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

3. 獲取和歸還鑽石對象：

```javascript
function getDiamond(x, y) {
  let diamond;
  if (activeDiamonds < DIAMOND_POOL_SIZE) {
    diamond = diamondPool[activeDiamonds];
    activeDiamonds++;
    diamond.style.display = 'block';
  } else {
    return null;
  }
  
  gsap.set(diamond, { 
    x: x - 30,
    y: y - 30,
    rotation: 0,
    scale: 1,
    opacity: 1
  });
  return diamond;
}

function returnDiamond(diamond) {
  diamond.style.display = 'none';
  activeDiamonds--;
}
```

4. 生成隨機終點位置：

```javascript
function getRandomPointInCircle(clickX, clickY, radius) {
  const angle = Math.random() * Math.PI;
  const r = radius * Math.sqrt(Math.random());
  const endX = clickX + r * Math.cos(angle);
  const endY = clickY - Math.abs(r * Math.sin(angle));
  return [endX, endY];
}
```

5. 動畫效果實現：

```javascript
function animateDiamond(diamond, clickX, clickY) {
  const [endX, endY] = getRandomPointInCircle(clickX, clickY, 100);
  const rotation = (Math.random() - 0.5) * 60; // 減少旋轉角度讓效果更自然
  const scale = 0.8 + Math.random() * 0.4;
  
  // 噴射階段
  gsap.to(diamond, {
    duration: 0.4 + Math.random() * 0.2,
    x: endX - 30,
    y: endY - 30,
    scale: scale,
    rotation: rotation,
    ease: "power2.out",
    onComplete: () => {
      // 下落階段
      gsap.to(diamond, {
        duration: 0.6 + Math.random() * 0.3,
        y: window.innerHeight + 50,
        rotation: rotation * 2,
        ease: "power1.in",
        opacity: 0,
        onComplete: () => {
          returnDiamond(diamond);
        }
      });
    }
  });
}
```

6. 事件監聽和效果觸發：

```javascript
// 初始化鑽石池
createDiamondPool();

// 效能優化：使用節流控制點擊頻率
let lastClickTime = 0;
const CLICK_THRESHOLD = 100;

document.addEventListener("click", (event) => {
  const now = Date.now();
  if (now - lastClickTime < CLICK_THRESHOLD) return;
  lastClickTime = now;

  // 固定產生DIAMOND_COUNT顆鑽石
  for (let i = 0; i < DIAMOND_COUNT; i++) {
    const diamond = getDiamond(event.clientX, event.clientY);
    if (diamond) {
      animateDiamond(diamond, event.clientX, event.clientY);
    }
  }
});
```

### 效能優化

為了提高效能，我們採用了以下策略：

1. 使用對象池減少 DOM 操作。
2. 應用 `will-change` CSS 屬性優化動畫性能。
3. 使用節流控制點擊頻率，避免過於頻繁的動畫觸發。

### 總結

通過結合 HTML、CSS 和 JavaScript，我們創建了一個互動式的鑽石噴灑效果。這個效果不僅視覺上吸引人，而且在性能方面也經過了優化。使用 GSAP 庫使得動畫更加流暢，而對象池的應用則提高了整體效率。這種技術可以應用於各種網頁遊戲或互動式網站，為用戶提供更加豐富的視覺體驗。
