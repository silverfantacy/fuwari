---
title: Proxyman 輕鬆解決前端開發 CORS 問題
published: 2024-08-24
description: 利用 Proxyman 的強大 Script 功能，輕鬆修改 API 返回的 Response header，徹底解決 CORS 限制。
image: ''
tags: [Proxyman, CORS, 跨域請求]
category: 前端
draft: false 
---

在前端開發過程中，經常會遇到 API 因為 CORS (Cross-Origin Resource Sharing，跨來源資源共用) 政策限制而無法正常存取的問題。這時，我們可以利用 [Proxyman](https://proxyman.io/) 這款強大的網路除錯工具，透過其 Script 功能，在 API 返回的 Response 中動態加入 CORS 相關的 Header，完美解決這個惱人的問題。

## 問題描述

當瀏覽器發出跨域請求時，如果伺服器沒有正確設定 CORS 相關的 Header，瀏覽器就會因為安全政策而阻擋這個請求，導致前端無法取得所需的資料。這在開發環境中尤其常見，因為本機伺服器與 API 伺服器的網域通常不同，很容易觸發 CORS 限制。

## 解決方案

Proxyman 提供了 Script 功能，讓我們可以在請求和回應之間加入客製化的邏輯。以下是一個簡單的範例，示範如何在 API 的 Response 中加入 CORS 相關的 Header：

```javascript
async function onResponse(context, url, request, response) {
  // ... (其他部分的程式碼)
  
  // Allow all
  response.headers["Access-Control-Allow-Origin"] = request.headers.Origin
  response.headers["Access-Control-Allow-Credentials"] = "true"
  
  // ... (其他部分的程式碼)

  return response;
}
```

這段程式碼會在每次 API 回應時執行。它會從 Request 的 Header 中取得 `Origin` 值，並將其設定到 Response 的 `Access-Control-Allow-Origin` Header 中。同時，它也會將 `Access-Control-Allow-Credentials` 設定為 `"true"`，允許瀏覽器在跨域請求時傳送 Cookies 等認證資訊。

## 為什麼不能直接設定 `Access-Control-Allow-Origin` 為 `*`？

雖然將 `Access-Control-Allow-Origin` 設定為 `*` 可以允許所有網域的請求，但在需要傳送 Cookies 等認證資訊的跨域請求中，瀏覽器會要求伺服器明確指定允許的網域，而不能使用萬用字元。同時，`Access-Control-Allow-Credentials` 也必須設定為 `"true"`，才能讓瀏覽器在跨域請求時傳送 Cookies。

> 參考資料：
> 
> [[Day 27] Cross-Origin Resource Sharing (CORS)](https://ithelp.ithome.com.tw/articles/10251693)