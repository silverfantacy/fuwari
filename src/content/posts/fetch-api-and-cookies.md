---
title: Fetch API 處理 Cookie 的跨域請求
published: 2024-10-19
description: 'Fetch API 在處理 Cookie 時的默認行為、如何在跨域請求中攜帶 Cookie。'
image: ''
tags: [Fetch API, Cookie, CORS, 跨域請求]
category: '前端'
draft: false 
---

## Fetch API 與 Cookie

### Fetch API 簡介

Fetch API 是現代前端開發中廣泛使用的網路請求工具，基於 Promise 設計，提供了簡潔的介面來處理 HTTP 請求和回應。與傳統的 XMLHttpRequest（XHR）相比，Fetch 更加靈活，易於與其他技術集成，如 Service Workers。

### 默認情況下的 Cookie 處理

默認情況下，Fetch API 不會發送或接收 Cookie。這是為了遵守同源政策（Same-Origin Policy）並增強網站的安全性。也就是說，除非明確配置，Fetch 請求不會攜帶跨站點的憑證資訊，包括 Cookie 和 HTTP 認證資訊。

### 如何在 Fetch 中攜帶 Cookie

要在 Fetch 請求中攜帶 Cookie，需要在請求的配置物件中設置 `credentials` 屬性為 `'include'`。這將確保請求包含跨站點的憑證資訊，包括 Cookie。

```javascript
fetch('https://example.com/api/data', {
  credentials: 'include'
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

### 跨域請求中的 Cookie

在跨域請求中攜帶 Cookie 需要伺服器端的配合。伺服器必須在回應標頭中設置適當的 CORS（跨域資源共享）標頭，以允許跨站點請求攜帶憑證資訊。具體來說，伺服器需要設置：

- `Access-Control-Allow-Origin` 為請求的來源。
- `Access-Control-Allow-Credentials` 為 `true`。

否則，瀏覽器將不會發送 Cookie，即使客戶端設置了 `credentials: 'include'`。
