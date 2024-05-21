---
title: 解決 sync 請求下的 Session 資料衝突
published: 2024-05-21
description: '當多個請求同時修改 Laravel 的 Session 資料時，可能會發生資料不一致的問題。'
image: 'https://media.virtualxnews.com/2024/05/ae1cfc77f1cc5ecf2d2436c41d75302c.png'
tags: [Laravel, Session]
category: '後端'
draft: false 
---
## Laravel Session 鎖定 (Session Blocking)：解決併發請求下的 Session 資料衝突

最近在開發網頁應用時，剛好遇到這樣的問題：

> 假設網頁需要同時取得 session 中的 A 資料並刪除 B 資料，由於 A 和 B 沒有關聯，你選擇用 JavaScript 同時發出兩個 API 請求，一個讀取 A，另一個刪除 B。
> 
> 然而，實際操作時卻發現，雖然 B 的資料看似被成功刪除，但再次讀取 session 時，B 的資料卻依然存在。這種情況很可能是因為同時讀取和刪除 session 資料造成了衝突。

這個問題的根源在於，Laravel 預設允許多個使用相同 session 的請求同時執行。這意味著，你的兩個請求可能在同一時間修改同一個 session，導致資料不一致。

### Laravel Session 鎖定 (Session Blocking)

為了解決這個問題，Laravel 提供了 Session 鎖定 (Session Blocking) 的機制。 Session 鎖定允許你限制特定 session 的併發請求數量。當一個請求有 session 鎖時，其他共享相同 session ID 的請求會被暫停，直到前一個請求完成。

如此一來，就能確保對同一個 session 的修改操作會依序進行，避免資料衝突。

### 如何使用 Session 鎖定？

要在 Laravel 中使用 Session 鎖定非常簡單，只需在路由定義中加上 `block` 方法：

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

在這個例子中，對 /profile (讀取 A 資料) 和 /order (刪除 B 資料) 的請求會取得 session 鎖定。當一個請求正在處理時 (例如正在讀取使用者資料)，另一個請求 (例如刪除訂單) 就會被暫時「卡住」，直到前一個請求完成釋放鎖定。

你可以根據需求調整 block 方法的參數：

* `$lockSeconds`：設定 Session 鎖定的最長時間。預設為 10 秒，但你可以根據預期處理時間調整。
* `$waitSeconds`：設定請求等待鎖定的最長時間。同樣預設為 10 秒，如果超過這個時間仍無法取得鎖定，系統會拋出 `Illuminate\Contracts\Cache\LockTimeoutException` 異常。

:::note
務必確認應用程式使用了支援「原子鎖定」([atomic locks](https://laravel.com/docs/master/cache#atomic-locks)) 的快取驅動程式，例如：`memcached`、`dynamodb`、`redis`、`database`、`file` 或 `array`。
不要使用 `cookie` 作為 Session 驅動程式，因為它不支援原子鎖定，無法確保 Session Blocking 的效果。
:::

> 參考資料：
> 
> [Session Blocking](https://laravel.com/docs/master/session#session-blocking)