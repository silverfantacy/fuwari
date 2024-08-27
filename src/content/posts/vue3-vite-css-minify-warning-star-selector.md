---
title: Vue3 + Vite 編譯 CSS 時出現「Expected identifier but found '*'」警告
published: 2024-08-27
description: 在使用 Vue3 和 Vite 開發時，如果遇到 CSS 壓縮過程中的「Expected identifier but found '*'」警告，可能原因和解決方案。
image: ''
tags: [Vue3, Vite, CSS, DataTables]
category: 前端
draft: false 
---

在使用 Vue3 和 Vite 編譯專案時，可能會在 CSS 壓縮階段遇到以下警告：

```
warnings when minifying css:
▲ [WARNING] Expected identifier but found "*" [css-syntax-error]

    <stdin>:4328:2:
      4328 │   *font-size: small;
           ╵   ^
```

深入分析該警告的成因，發現源自專案中引入的舊版 DataTables。這些舊版本的 DataTables CSS 檔案中，可能包含了諸如 *cursor: hand 等專為早期 Internet Explorer 設計的 CSS hack。這些 hack 在現代瀏覽器和構建工具中已經不再適用，甚至可能導致 CSS 壓縮過程中的衝突。

## 解決方案

最直接的解決方案是更新 [DataTables](https://datatables.net/) 套件到最新版本。新版本的 [DataTables](https://cdn.datatables.net/2.1.4/css/dataTables.dataTables.css) 應該已經修正了這個 CSS 語法。

如果你無法立即更新 DataTables，或者問題不是由 DataTables 引起的，可以嘗試以下步驟：

1. **仔細檢查 CSS 檔案**： 找到出現警告訊息的 CSS 檔案，並檢查是否有類似 `*屬性名稱: 值;` 的錯誤語法。如果有，將星號移除或修正為正確的 CSS 語法。
2. **暫時禁用 CSS 壓縮**： 在開發過程中，你可以暫時禁用 Vite 的 CSS 壓縮功能，以便更輕鬆地調試問題。你可以在 `vite.config.js` 檔案中設定 `build.cssMinify` 為 `false` 來禁用 CSS 壓縮。


> 參考資料：
> 
> [Expected identifier but found "*" [css-syntax-error] while updating angular 10 to 14](https://stackoverflow.com/questions/76952838/expected-identifier-but-found-css-syntax-error-while-updating-angular-10-t)