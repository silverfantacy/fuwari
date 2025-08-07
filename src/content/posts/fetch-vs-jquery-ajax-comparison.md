---
title: "fetch 與 $.ajax 深度比較：為什麼 fetch 容易遇到 CORS 問題？"
published: 2025-08-07
description: "詳細分析 fetch API 和 jQuery $.ajax 的差異，探討為什麼 fetch 在跨域請求中更容易遇到 CORS 問題，以及如何選擇適合的請求方法"
image: ""
tags: ["JavaScript", "CORS", "fetch", "jQuery", "前端開發"]
category: "前端"
draft: false
---

# fetch 與 $.ajax 深度比較：為什麼 fetch 容易遇到 CORS 問題？

在現代前端開發中，HTTP 請求是不可或缺的功能。雖然 `fetch` API 是現代瀏覽器的原生標準，但許多開發者發現它在處理跨域請求時比 jQuery 的 `$.ajax` 更容易遇到問題。本文將深入探討這兩種方法的差異，幫助你選擇最適合的解決方案。

## 主要差異比較

### 1. Cookie 處理方式

這是兩者最關鍵的差異之一：

```javascript
// fetch 寫法 - 需要明確指定
fetch(url, {
  credentials: 'include'  // 需要明確指定才會發送 cookies
})

// $.ajax 寫法 - jQuery 的設定方式
$.ajax({
  xhrFields: {
    withCredentials: true  // jQuery 的 cookie 設定方式
  }
})
```

**關鍵差異：**
- `fetch` 預設不會發送 cookies，必須明確設定 `credentials: 'include'`
- `$.ajax` 可以透過 `withCredentials` 自動處理認證相關的 cookies

### 2. 瀏覽器相容性

```javascript
// fetch: 現代瀏覽器 API
if ('fetch' in window) {
  // 支援 fetch
} else {
  // 需要 polyfill 或降級到 XMLHttpRequest
}

// $.ajax: jQuery 封裝的 XMLHttpRequest
// 相容性更好，支援所有瀏覽器
```

### 3. 預設行為比較

#### fetch 的預設行為
```javascript
// fetch 預設行為
fetch('/api/data')
  .then(response => {
    // ❌ 需要手動檢查狀態
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    // ❌ 需要手動轉換 JSON
    return response.json()
  })
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error))
```

#### $.ajax 的預設行為
```javascript
// $.ajax 預設行為
$.ajax({
  url: '/api/data',
  dataType: 'json',    // ✅ 自動解析 JSON
  success: function(data) {
    // ✅ 只有成功才會執行
    console.log(data)
  },
  error: function(xhr, status, error) {
    // ✅ 自動處理 HTTP 錯誤狀態
    console.error('Error:', error)
  }
})
```

## 為什麼 fetch 容易遇到 CORS 問題？

### 1. 更嚴格的安全政策

```javascript
// fetch 對跨域請求更嚴格
fetch(url, {
  credentials: 'include',  // 必須明確指定
  mode: 'cors',           // 明確的 CORS 模式
  headers: {
    'Content-Type': 'application/json',
  }
})
```

`fetch` 採用更嚴格的安全模型，要求開發者明確處理每個安全相關的設定。

### 2. Cookie 發送機制

```javascript
// ❌ fetch: 預設不發送 cookies
fetch('/api/data')  // 不會發送 cookies，可能導致認證失敗

// ✅ $.ajax: 根據 jQuery 版本和設定會自動處理
$.ajax({
  url: '/api/data'  // 可能會自動發送必要的 cookies
})
```

### 3. 錯誤處理差異

```javascript
// fetch: 只有網路錯誤才會 reject
fetch('/api/data')
  .then(response => {
    // ❌ HTTP 404, 500 等狀態碼不會自動拋出錯誤
    if (!response.ok) {
      throw new Error('HTTP error')
    }
    return response.json()
  })
  .catch(error => {
    // 只處理網路錯誤和手動拋出的錯誤
    console.error(error)
  })

// $.ajax: 自動處理 HTTP 錯誤狀態
$.ajax({
  url: '/api/data'
})
.done(function(data) {
  // ✅ 只有 2xx 狀態碼才會執行
  console.log(data)
})
.fail(function(xhr) {
  // ✅ HTTP 錯誤會自動進入這裡
  console.error('HTTP error:', xhr.status)
})
```

## 實際專案中的應用範例

### 原本的 fetch 版本

```javascript
const response = await fetch('https://api.example.com/v1/users/123', {
  method: 'GET',
  credentials: 'include',  // 明確指定發送 cookies
  headers: {
    'Content-Type': 'application/json',
  },
});

// ❌ 需要手動檢查狀態
if (!response.ok) {
  throw new Error(`HTTP error! status: ${response.status}`);
}

// ❌ 手動解析 JSON
const result = await response.json();
```

### 修改後的 $.ajax 版本

```javascript
const result = await new Promise((resolve, reject) => {
  $.ajax({
    url: 'https://api.example.com/v1/users/123',
    type: 'GET',
    dataType: 'json',    // ✅ 自動解析 JSON
    xhrFields: {
      withCredentials: true  // ✅ jQuery 的 cookie 設定方式
    }
  })
  .done(function(response) {
    // ✅ 自動處理成功狀態
    resolve(response);
  })
  .fail(function(xhr, status, error) {
    // ✅ 自動處理錯誤狀態
    reject(new Error(`HTTP error! status: ${xhr.status}, error: ${error}`));
  });
});
```

## 為什麼選擇 $.ajax？

在某些情況下，選擇 `$.ajax` 可能更適合：

### 1. 與專案架構一致
如果專案已經大量使用 jQuery，保持一致性能減少複雜性。

### 2. 更好的 Cookie 處理
`$.ajax` 對認證相關的 cookies 處理更成熟。

### 3. 減少 CORS 問題
jQuery 的處理方式對跨域請求更友善。

### 4. 錯誤處理更簡單
自動區分成功和失敗狀態，減少手動檢查。

## 現代化的解決方案

### 封裝 fetch 以獲得更好的體驗

```javascript
// 創建一個封裝函數
async function apiRequest(url, options = {}) {
  const defaultOptions = {
    credentials: 'include',
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
    ...options,
  };

  try {
    const response = await fetch(url, defaultOptions);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('API Request failed:', error);
    throw error;
  }
}

// 使用封裝函數
const result = await apiRequest('https://api.example.com/v1/users/123');
```

### 使用現代的 HTTP 客戶端庫

```javascript
// 使用 axios（推薦）
import axios from 'axios';

const api = axios.create({
  withCredentials: true,
  headers: {
    'Content-Type': 'application/json',
  },
});

const result = await api.get('https://api.example.com/v1/users/123');
```

## 總結

| 特性 | fetch | $.ajax | 推薦使用場景 |
|------|-------|--------|-------------|
| 現代化 | ✅ 原生 API | ❌ 需要 jQuery | 新專案 |
| 學習曲線 | 較陡峭 | 較平緩 | 團隊經驗 |
| 錯誤處理 | 需手動處理 | 自動處理 | 快速開發 |
| Cookie 處理 | 需明確設定 | 相對自動 | 認證相關 |
| 檔案大小 | 無額外負擔 | 需要 jQuery | 效能考量 |
| CORS 相容性 | 較嚴格 | 較寬鬆 | 跨域需求 |

**建議：**
- **新專案**：使用 `fetch` 配合適當的封裝
- **現有 jQuery 專案**：保持 `$.ajax` 的一致性
- **複雜認證需求**：考慮使用 `axios` 等成熟庫
- **學習目的**：理解 `fetch` 的原理，掌握現代標準

`fetch` 是未來趨勢，但在處理認證和跨域請求時需要更多手動設定。而 `$.ajax` 經過多年發展，在這些方面有更成熟的預設行為。選擇哪種方法取決於你的專案需求、團隊經驗和長期維護考量。

> 參考資料：
>
> [MDN - Fetch API](https://developer.mozilla.org/zh-TW/docs/Web/API/Fetch_API)
>
> [jQuery.ajax() | jQuery API Documentation](https://api.jquery.com/jquery.ajax/)
>
> [MDN - CORS](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)