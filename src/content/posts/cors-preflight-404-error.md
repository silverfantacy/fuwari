---
title: 解決 CORS Preflight 404 錯誤
published: 2025-05-06
description: '深入解析「Failed to load resource: Preflight response is not successful. Status code: 404」錯誤的原因與解決方案'
image: ''
tags: [CORS, 跨域請求, Preflight, OPTIONS, 網頁開發]
category: '前端'
draft: false 
---

## CORS Preflight 404 錯誤解析

在前端開發中，你可能會遇到以下錯誤訊息：

```
Failed to load resource: Preflight response is not successful. Status code: 404
```

這個錯誤表示你的跨來源資源共享（CORS）請求中，瀏覽器自動發起的預檢（Preflight）請求失敗了，伺服器回應了 HTTP 404 狀態碼。這是一個常見但經常被誤解的問題，本文將詳細解釋其原因和解決方法。

## 什麼是 Preflight 請求？

當瀏覽器需要進行「非簡單請求」的跨域操作時，會先發送一個 OPTIONS 方法的「預檢請求」（Preflight Request）到伺服器，以確認伺服器是否允許實際的請求。這是 CORS 機制的一部分，目的是保護伺服器不受未經授權的跨域請求影響。

以下情況會觸發預檢請求：

1. 使用 GET、HEAD、POST 以外的 HTTP 方法（如 PUT、DELETE）
2. 設置了特定的自定義請求標頭（如 `Authorization`、`X-API-Key` 等）
3. 使用 `Content-Type` 為 `application/json` 等非簡單內容類型
4. 使用了 `credentials: 'include'` 或 `withCredentials: true` 設置

## 404 錯誤的原因

當預檢請求收到 404 回應時，表示：

1. **伺服器未處理 OPTIONS 請求**：伺服器可能沒有配置來回應 OPTIONS 方法，或者未在請求的路徑上正確處理該方法。
2. **URL 路徑錯誤**：請求的 URL 可能不存在或拼寫錯誤，導致伺服器找不到對應的資源。
3. **伺服器配置問題**：伺服器可能缺少處理 CORS 的中間件或配置。
4. **路由處理不正確**：後端框架可能未正確設置處理 OPTIONS 請求的路由。

## 解決方案

### 1. 前端解決方案

#### 簡化請求（如果可能）

如果可以，將請求簡化為「簡單請求」以避免預檢：

- 使用 GET、HEAD 或 POST 方法
- 移除自定義標頭
- 使用 `application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain` 作為 Content-Type

```javascript
// 替代
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: new URLSearchParams({
    key: 'value'
  })
})
```

#### 開發環境使用代理

在開發環境中設置代理伺服器可以繞過 CORS 限制：

```json
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
      },
    }
  }
}
```

#### 使用 CORS 代理工具

對於測試或開發，可以使用如 [Proxyman](https://proxyman.io/) 等工具修改 API 回應標頭：

```javascript
async function onResponse(context, url, request, response) {
  // 為所有回應添加 CORS 標頭
  response.headers["Access-Control-Allow-Origin"] = request.headers.Origin
  response.headers["Access-Control-Allow-Credentials"] = "true"
  response.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE, OPTIONS"
  response.headers["Access-Control-Allow-Headers"] = "Content-Type, Authorization"
  
  return response;
}
```

### 2. 後端解決方案

#### Express.js

```javascript
const express = require('express');
const app = express();

// CORS 中間件
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://your-frontend-domain.com');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  
  // 處理 OPTIONS 請求
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  
  next();
});

// 或使用 cors 套件
const cors = require('cors');
app.use(cors({
  origin: 'https://your-frontend-domain.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}));
```

#### 其他後端框架

根據你使用的後端框架，確保：

1. 正確配置 CORS 標頭
2. 顯式處理 OPTIONS 請求
3. 在所有 API 路由上允許 OPTIONS 方法

## 最佳實踐

1. **明確配置 CORS**：不要使用 `Access-Control-Allow-Origin: *` 當你需要攜帶憑證（cookies）時
2. **檢查請求 URL**：確保 URL 正確，並且後端確實有對應的路由
3. **查看網路請求**：使用瀏覽器開發者工具中的網路面板來查看具體的預檢請求細節
4. **測試簡化的請求**：嘗試簡化請求，看錯誤是否與特定請求設置有關
5. **後端日誌**：檢查伺服器日誌以確認是否收到了 OPTIONS 請求

## 常見錯誤排除

### 情境一：使用 Axios 發送 PUT 請求時出現 404

```javascript
axios.put('https://api.example.com/resource', data, {
  withCredentials: true
})
```

**問題**：這會觸發預檢請求，但後端可能未處理 OPTIONS 方法。

**解決方案**：確保後端路由處理 OPTIONS 請求。

### 情境二：在 React 專案中使用 fetch API 發送 JSON 資料

```javascript
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
})
```

**問題**：Content-Type 為 application/json 會觸發預檢請求。

**解決方案**：後端需設置正確的 CORS 標頭。

## 總結

CORS Preflight 404 錯誤通常是由於伺服器未正確處理 OPTIONS 請求導致的。解決方法包括：

1. 確保後端正確處理 OPTIONS 請求
2. 設置適當的 CORS 標頭
3. 在開發環境中使用代理服務器
4. 簡化請求（如果可能）

透過理解 CORS 機制和預檢請求的運作原理，你將能夠更有效地診斷和解決這類問題，讓你的前端應用能夠順利與其他來源的資源進行互動。

## 參考資源

1. [MDN Web Docs: CORS](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS) - Mozilla 開發者網路上關於跨來源資源共享的完整指南
2. [CORS 完全手冊（一）：為什麼會發生 CORS 錯誤？](https://blog.huli.tw/2021/02/19/cors-guide-1/) - Huli 的詳細 CORS 指南

---

希望這篇文章對你有所幫助！如果你有進一步的問題或需要更深入的解釋，歡迎在下方留言。 