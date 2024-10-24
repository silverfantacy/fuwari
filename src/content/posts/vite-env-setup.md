---
title: Vite 環境變數 .env 的配置及使用
published: 2024-10-24
description: '如何在 Vite 中配置和使用 .env 文件來管理環境變數。'
image: ''
tags: [Vite, 環境變數, .env, 前端開發]
category: '前端'
draft: false 
---

## 什麼是環境變數?

環境變數是一種用來在不同環境中配置應用程式的方式。在前端開發中,我們經常需要針對不同的環境(開發、測試、生產)設置不同的配置,例如:

- API 的請求地址
- 調試模式開關
- 運行環境標識
- 其他環境相關的配置

## Vite 中的環境變數配置

### 基本配置方式

1. **創建環境變數文件**

在專案根目錄創建 `.env` 文件:

```bash
# .env
VITE_API_URL=https://api.example.com
VITE_DEBUG_MODE=true
VITE_APP_TITLE=My App
```

2. **命名規則**

- 必須以 `VITE_` 開頭
- 只有 `VITE_` 開頭的變數才會暴露給客戶端代碼

3. **在代碼中使用**

```javascript
console.log(import.meta.env.VITE_API_URL)
console.log(import.meta.env.VITE_APP_TITLE)
```

### 進階配置

1. **區分環境**

可以創建不同的環境文件:

```bash
.env                # 所有環境都會載入
.env.development    # 開發環境
.env.production     # 生產環境
```

2. **使用 loadEnv 函數**

在 `vite.config.js` 中可以使用 `loadEnv` 函數來加載環境變數:

```javascript
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  // 加載指定模式下的環境變數
  const env = loadEnv(mode, process.cwd(), '')
  
  return {
    // 在配置中使用環境變數
    define: {
      'process.env.VITE_API_URL': JSON.stringify(env.VITE_API_URL),
      'process.env.VITE_DEBUG_MODE': JSON.stringify(env.VITE_DEBUG_MODE),
    },
    server: {
      port: parseInt(env.VITE_PORT) || 3000,
      proxy: {
        '/api': {
          target: env.VITE_API_URL,
          changeOrigin: true,
        }
      }
    },
    // 其他配置...
  }
})
```

### loadEnv

`loadEnv` 函數是 Vite 提供的實用工具,用於加載環境變數。它有三個參數:

1. **mode**: 當前的運行模式(development/production)
2. **envDir**: 環境變數文件所在的目錄(通常是項目根目錄)
3. **prefix**: 環境變數的前綴(默認是 'VITE_')

## 生產環境建構 (Production Build)

在使用 `yarn build` 進行生產環境建構時，我們可以通過以下方式來使用不同的環境變數：

### 1. 使用 --mode 參數

```bash
# 使用 staging 環境配置進行建構
yarn build --mode staging

# 使用 production 環境配置進行建構
yarn build --mode production
```

### 2. 在 package.json 中配置腳本

```json
{
  "scripts": {
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production",
    "build:test": "vite build --mode test"
  }
}
```

然後可以執行：
```bash
yarn build:staging  # 使用 .env.staging 配置
yarn build:prod     # 使用 .env.production 配置
yarn build:test     # 使用 .env.test 配置
```

### 3. 使用 cross-env 設置環境變數

如果需要在建構時設置額外的環境變數，可以使用 cross-env：

```json
{
  "scripts": {
    "build:staging": "cross-env NODE_ENV=staging vite build --mode staging",
    "build:prod": "cross-env NODE_ENV=production vite build --mode production"
  }
}
```

### 環境變數加載優先順序

根據 [Vite 官方文檔](https://vitejs.dev/guide/env-and-mode.html)，環境變數的加載優先順序為：

1. 命令行中設置的環境變數（最高優先級）
2. `.env.[mode].local`
3. `.env.[mode]`
4. `.env.local`
5. `.env`

### 在 vite.config.js 中使用環境變數

```javascript
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ command, mode }) => {
  // 加載環境變數
  const env = loadEnv(mode, process.cwd(), '')
  
  return {
    // 在配置中使用環境變數
    define: {
      'process.env.VITE_API_URL': JSON.stringify(env.VITE_API_URL),
    },
    build: {
      outDir: env.VITE_OUTPUT_DIR || 'dist',
      // 其他建構配置...
    }
  }
})
```

這樣的配置可以讓你在不同環境下靈活地使用不同的環境變數進行建構。
