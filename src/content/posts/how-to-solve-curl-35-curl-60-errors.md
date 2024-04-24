---
title: 解決 cURL 35 和 cURL 60 錯誤的方法
published: 2024-04-24
description: 解決 cURL 35 和 cURL 60 錯誤的步驟，包括確認 OpenSSL 版本、設定 OpenSSL 路徑、更新 CA 憑證以及重啟 PHP 和 Valet 服務。
image: 'https://media.virtualxnews.com/2024/04/b54c0811266ac80d6d327db773600e6f.jpg'
tags: [cURL, SSL, OpenSSL, PHP, Valet, Laravel]
category: 後端
draft: false 
---

本機開發 Laravel 的過程中，遇到錯誤訊息 `cURL 35` 或 `cURL error 60: SSL certificate problem: unable to get local issuer cert ificate` 的處理方法。

```bash
// 錯誤訊息大致如下
cURL error 35: OpenSSL/1.1.1s: error:0407008A:rsa routines:RSA_padding_check_PKCS1_type_1:invalid padding (see https://curl.haxx.se/libcurl/c/libcurl-errors.html) for https://xxxx.test/api/v1/xxx/xxx
```

解決 `cURL 35` 和 `cURL error 60` 的步驟大致相同。

## 步驟 1：確認 OpenSSL 版本
需要確認系統上安裝的 OpenSSL 版本。可以使用以下命令來檢查：

```bash
openssl version
```

確保你的 OpenSSL 是最新版本。如果不是最新版本，可以使用以下命令來更新：

```bash
brew upgrade openssl@1.1
```

或者

```bash
brew upgrade openssl@3
```

## 步驟 2：找到 php.ini 檔案
接下來，我們需要找到 php.ini 檔案的位置。可以使用以下命令來找到當前的 php.ini 檔案位置：

```bash
php -i | grep .ini
```

將找到的 php.ini 檔案位置備用。

## 步驟 3：設定 OpenSSL 路徑
在 php.ini 檔案中，找到以下設定並確保路徑正確：

對於 OpenSSL 1.1 版本：

```bash
openssl.cafile = "/opt/homebrew/etc/openssl@1.1/cert.pem"
```

對於 OpenSSL 3 版本：

```bash
openssl.cafile = "/opt/homebrew/etc/openssl@3/cert.pem"
```

請根據你的 OpenSSL 版本選擇正確的設定，並將其添加到 php.ini 檔案中。

## 步驟 4：更新 CA 憑證
接下來，我們需要將 `~/.config/valet/CA/LaravelValetCASelfSigned.pem` 的 CA 金鑰複製到 `/opt/homebrew/etc/openssl@{Version}/cert.pem` 檔案的最底部。這樣可以確保 cURL 可以正確驗證 SSL 憑證。

## 步驟 5：重啟 PHP 和 Valet
最後，我們需要重啟 PHP 和 Valet 服務以應用這些變更。可以使用以下命令來重啟：

```bash
brew services restart php && valet restart
```

這樣就完成了！應該能夠解決 `cURL 35` 和 `cURL error 60` 的錯誤了。