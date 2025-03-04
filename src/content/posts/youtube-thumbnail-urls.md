---
title: YouTube 影片取得縮圖
published: 2025-03-04
description: '介紹如何取得不同解析度的 YouTube 影片縮圖，包含完整的網址格式說明和實用的 JavaScript 工具函數，幫助開發者輕鬆整合 YouTube 縮圖到自己的專案中。'
image: 'https://imager.virtualxnews.com/rest/heWgjPK.jpeg'
tags: ['前端開發', 'JavaScript']
category: '前端'
draft: false 
---

YouTube 提供多種不同解析度的影片縮圖，只要替換網址中的影片 ID (xxxxxx) 即可取得。

## 縮圖網址格式

- 高解析度大圖 (1280 × 720)
`http://img.youtube.com/vi/xxxxxx/maxresdefault.jpg`
- 標準清晰圖 (640 × 480)
`http://img.youtube.com/vi/xxxxxx/sddefault.jpg`
- 高品質縮圖 (480 × 360)
`https://img.youtube.com/vi/xxxxxx/hqdefault.jpg`
- 播放器背景縮圖 (480 × 360)
`http://img.youtube.com/vi/xxxxxx/0.jpg`
- 影片開始畫面縮圖 (120 × 90)
`http://img.youtube.com/vi/xxxxxx/1.jpg`
- 影片中間片段縮圖 (120 × 90)
`http://img.youtube.com/vi/xxxxxx/2.jpg`
- 影片結束縮圖 (120 × 90)
`http://img.youtube.com/vi/xxxxxx/3.jpg`

使用方法：將網址中的 xxxxxx 替換為目標 YouTube 影片的 ID 即可。

EX：

https://img.youtube.com/vi/ZmzXkTHtPjA/maxresdefault.jpg

https://img.youtube.com/vi/t12bgcn01kE/maxresdefault.jpg

## YouTube 影片資訊擷取工具

以下是一個實用的 JavaScript 函數，能夠從 YouTube 一般影片或 Shorts 短片的 URL 中提取影片 ID、類型，並生成高解析度縮圖網址：

```jsx
function getYouTubeInfo(url) {
  // 定義正則表達式以匹配 YouTube URL 和 Shorts URL
  const youtubeRegex = /^(?:https?:\/\/)?(?:www\.)?(?:youtu\.be\/|youtube\.com\/(?:embed\/|v\/|watch\?v=|watch\?.+&v=))((\w|-){11})(?:\S+)?$/;
  const shortsRegex = /^(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/shorts\/)([\w-]{11})(?:\S+)?$/;

  let videoId = null;
  let type = null;

  // 判斷是否為 Shorts 連結
  let shortsMatch = url.match(shortsRegex);
  if (shortsMatch) {
    videoId = shortsMatch[1];
    type = "shorts";
  } else {
    // 判斷是否為 YouTube 連結
    let youtubeMatch = url.match(youtubeRegex);
    if (youtubeMatch) {
      videoId = youtubeMatch[1];
      type = "youtube";
    }
  }

  // 如果不是有效的 YouTube 或 Shorts 連結，返回 false
  if (!videoId) {
    return false;
  }

  // 組合封面圖片網址
  const thumbnailUrl = `http://img.youtube.com/vi/${videoId}/maxresdefault.jpg`;

  return {
    id: videoId,
    type: type,
    thumbnail: thumbnailUrl,
  };
}

// 測試範例
console.log(getYouTubeInfo("https://www.youtube.com/watch?v=dQw4w9WgXcQ"));
// 輸出: { id: "dQw4w9WgXcQ", type: "youtube", thumbnail: "http://img.youtube.com/vi/dQw4w9WgXcQ/maxresdefault.jpg" }

console.log(getYouTubeInfo("https://youtu.be/dQw4w9WgXcQ"));
// 輸出: { id: "dQw4w9WgXcQ", type: "youtube", thumbnail: "http://img.youtube.com/vi/dQw4w9WgXcQ/maxresdefault.jpg" }

console.log(getYouTubeInfo("https://youtube.com/shorts/0dPkkQeRwTI?feature=share"));
// 輸出: { id: "0dPkkQeRwTI", type: "shorts", thumbnail: "http://img.youtube.com/vi/0dPkkQeRwTI/maxresdefault.jpg" }

console.log(getYouTubeInfo("https://www.example.com"));
// 輸出: false
```

這個函數不僅能識別標準的 YouTube 網址，還支援短連結和 YouTube Shorts 格式，十分適合需要處理 YouTube 內容的前端應用。