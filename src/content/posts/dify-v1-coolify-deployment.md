---
title: 使用 Coolify 部署 Dify v1.1.1 紀錄
published: 2025-03-14
description: '記錄如何使用 Coolify 快速部署 Dify v1.1.1，包含常見問題處理與最佳設定，輕鬆架設自己的 AI 應用開發平台。'
image: 'https://imager.virtualxnews.com/rest/f6VKeoK.png'
tags: ['Dify', 'Coolify', '自架伺服器', 'AI 平台']
category: 'AI'
draft: false 
---

# 使用 Coolify 部署 Dify v1.1.1 紀錄

> **緊急更新 (2025/03/21)**：發現安裝 plugin 後重啟服務時，會發生 Redis 無法啟動的錯誤。如果遇到這個問題，需要修改 docker-compose 檔案中的以下設定：
> 
> 將原本的：
> ```yaml
> plugin_daemon:
>   volumes:
>     - 'dify-storage:/app/storage'
> ```
> 
> 改為：
> ```yaml
> plugin_daemon:
>   volumes:
>     - 'dify-storage-plugin:/app/storage'
> ```
> 
> 這樣應該可以解決服務無法正常啟動的問題。

最近 [Dify](https://dify.ai) 正式發布正式 v1.1.1 版本，作為一個開源的 LLMOps 平台，它提供了從 AI 應用搭建到部署的完整解決方案。而 [Coolify](https://coolify.io) 作為一個自託管的 Heroku/Netlify/Vercel 替代品，提供了簡單但強大的部署方式。

## 特別建議：原本使用 Dify v0.15.3 進行全新安裝

## 前置準備

在開始前，請確保您有：

1. 一台已安裝 Coolify 的伺服器 (如尚未安裝可參考[官方文件](https://coolify.io/docs/installation))
2. 接近 4GB 以上的可用記憶體 (Dify 全功能運行建議)
3. 基本的 Docker 和網路知識

## 部署步驟

### 1. 使用 docker-compose 部署 Dify 服務

根據 Coolify PR ([#5320](https://github.com/coollabsio/coolify/pull/5320)) 中的設定，部署 Dify 最簡單的方式是直接使用 docker-compose 檔案：

1. 登入您的 Coolify 控制台
2. 建立新服務 → 選擇「Docker Compose」服務類型
3. 上傳或貼上基於 PR #5320 的 docker-compose.yml 檔案

### 2. 設定必要參數與環境變數

在 docker-compose 檔案中，需要注意以下幾個重要參數設定：

```bash
# PostgreSQL 設定
# 將 image: 'postgres:15-alpine' 修改為
image: 'postgres:16-alpine'  # 提供更好的效能與安全性

# 核心功能配置
SECRET_KEY=your-secret-key  # 建議生成一個隨機字串
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-db-password
POSTGRES_DB=dify

# 文件上傳限制設定 (重要)
UPLOAD_FILE_SIZE_LIMIT=50M
UPLOAD_FILE_BATCH_LIMIT=10
UPLOAD_IMAGE_FILE_SIZE_LIMIT=15M
NGINX_CLIENT_MAX_BODY_SIZE=50M  # 特別注意這項設定，必須在部署前設定
```

> **重要提示**：`NGINX_CLIENT_MAX_BODY_SIZE` 必須優先設定，它決定了上傳檔案的大小限制。根據您的實際需求，可能需要調整為更大的值，特別是當您計劃上傳大型 PDF 或其他文件到知識庫時。

### 3. 配置持久化存儲

Dify 的數據需要持久化存儲，請在 docker-compose 檔案中確保配置以下存儲卷：

```yaml
volumes:
  - ./data/storage:/app/api/storage  # 存儲上傳的檔案和索引
  - ./data/postgres:/var/lib/postgresql/data  # PostgreSQL 數據
  - ./data/tasks:/app/api/tasks  # 任務處理
```

### 4. 網路和安全設定

在 Coolify 控制台中，完成以下網路設定：

1. 域名配置：
   - 設定您的自訂域名 (如 `dify.yourcompany.com`)
   - 啟用 HTTPS (強烈建議)

2. 防火牆設定：
   - 確保開放 TCP 80 和 443 連接埠
   - 如使用內部網路，請配置相應的網路規則

### 5. 資源配置

根據預期使用情況調整資源限制：

- 基本使用: 2 CPU / 2GB RAM
- 推薦配置: 4 CPU / 4GB RAM (用於大型索引和多用戶)

### 6. 啟動和驗證

1. 完成所有配置後，點擊「部署」按鈕
2. 等待服務啟動 (初次部署可能需要 3-5 分鐘)
3. 部署成功後，訪問配置的域名
4. 首次訪問會引導您創建管理員帳號

## 常見問題與解決方案

### 文件上傳問題

如果遇到文件上傳失敗，通常與 NGINX 配置有關：

```bash
# 確保在部署前已正確設定這個變數
NGINX_CLIENT_MAX_BODY_SIZE=50M  # 或更大值，取決於您的需求
```

### 數據庫連線問題

如果出現數據庫連線錯誤，請檢查：

1. PostgreSQL 環境變數是否正確設定
2. 數據庫存儲卷是否正確掛載
3. 服務啟動日誌中有無具體錯誤信息

> **PostgreSQL 版本說明**：PostgreSQL 16-alpine 相比於 15-alpine 提供更好的效能、更新的功能以及安全更新，強烈建議在新部署中使用。

### 效能優化

對於生產環境，推薦添加以下配置來優化效能：

```bash
# 增加 Web 服務工作進程
WEB_CONCURRENCY=2

# 知識庫處理設定
CELERY_WORKER_CLASS=prefork
MAX_UPLOAD_WORKERS=2

# Redis 優化
REDIS_QUEUE_MAX_CONCURRENCY=10
```

## 結論

希望這份部署紀錄能幫助大家順利架設自己的 Dify 平台。

> 相關資源：
> - [Dify 官方文件](https://docs.dify.ai/)
> - [Coolify 官方文件](https://coolify.io/docs/)
> - [相關 PR #5320](https://github.com/coollabsio/coolify/pull/5320)