---
title: Dify 環境設定與部署
published: 2025-02-19
description: '介紹 Dify 重要環境設定與部署方式，包含知識庫設定、系統限制調整'
image: 'https://imager.virtualxnews.com/rest/f6VKeoK.png'
tags: [Dify, AI, No-code, Zeabur]
category: 'AI'
draft: false 
---

# Dify 環境設定與部署指南

這篇是我在部署 Dify 時整理的一些重要環境設定筆記，主要記錄一些容易被忽略但實務上很重要的設定值，方便日後查閱與參考。

## 核心功能設定

### 知識庫分段長度設定

分段長度對於知識庫的處理效能有重大影響，建議調整：

```bash
INDEXING_MAX_SEGMENTATION_TOKENS_LENGTH = 4000
```

> 注意：雖然建議值為 50-1000，但根據實際測試，設定更大的值可以提升長文本的處理效果。

## 系統限制設定

### 檔案處理限制
```bash
# Dify 環境變數
UPLOAD_FILE_SIZE_LIMIT=50M    # 單檔上傳限制，預設 15M
UPLOAD_FILE_BATCH_LIMIT=10    # 批次上傳數量，預設 5
UPLOAD_IMAGE_FILE_SIZE_LIMIT=15M  # 圖片上傳限制

# Nginx 配置
NGINX_CLIENT_MAX_BODY_SIZE=50M
```

## 最佳部署方式

### Zeabur 一鍵部署
[![Deployed on Zeabur](https://zeabur.com/deployed-on-zeabur-dark.svg)](https://zeabur.com?referralCode=silverfantacy&utm_source=silverfantacy)

強烈推薦使用 [Zeabur](https://zeabur.com/templates/1D4DOW) 部署 Dify：

- 一鍵部署：模板化部署流程
- 自動擴展：智能資源調配
- 簡易維護：直觀的管理界面
- 穩定可靠：企業級服務品質

### 部署步驟：
1. 進入 [Zeabur Dify 模板頁面](https://zeabur.com/templates/1D4DOW)
2. 點擊部署並登入帳號
3. 設定環境變數：
   - 必要設定：API Keys、資料庫連接等
   - 選填設定：上述提到的進階設定
4. 啟動服務並等待部署完成


> 參考資料：
> - [Dify 官方文檔](https://docs.dify.ai/zh-hans/getting-started/install-self-hosted/environments)