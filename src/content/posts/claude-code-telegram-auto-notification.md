---
title: "Claude Code 完成任務自動 Telegram 通知：再也不用一直盯著螢幕了"
published: 2025-07-16
description: "用 Claude Code 跑任務總是不知道什麼時候完成，經常忘記？設定個 Telegram 通知，任務完成就推播給你！"
image: ""
tags: ["Claude Code", "MCP", "Telegram", "Cloudflare Workers", "自動化", "通知系統"]
category: "AI"
draft: false
---

# Claude Code 完成任務自動 Telegram 通知：再也不用一直盯著螢幕了

你是不是也有這個困擾？用 Claude Code 跑任務的時候，不知道什麼時候會完成，結果要嘛一直盯著螢幕，要嘛就是忘記了，回來才發現早就跑完了。

我最近發現一個超實用的工具 `telegram-notification-mcp`，可以讓 Claude Code 完成任務的時候自動傳 Telegram 訊息通知你。設定完之後，你就可以放心去做別的事情，任務完成就會收到通知，超方便的！

## 這個工具長什麼樣？

[telegram-notification-mcp](https://github.com/kstonekuan/telegram-notification-mcp) 就是一個讓 Claude Code 可以傳 Telegram 訊息的小工具。簡單說，它會讓 Claude Code 在任務完成、發生錯誤、或是需要你介入的時候，自動傳一則訊息到你的 Telegram。

最好的是，它可以用 Cloudflare Workers 部署，完全不用自己架伺服器，也不用擔心維護問題。設定一次就可以一直用下去。

## 怎麼設定？

### 第一步：建立 Telegram Bot

在 Telegram 中搜尋 `@BotFather`，然後：

1. 發送 `/newbot` 命令
2. 設定 bot 名稱和用戶名
3. 取得 `BOT_TOKEN`（長這樣：123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11）

### 第二步：取得你的 Chat ID

最簡單的方法是在 Telegram 中搜尋 `@userinfobot`，點擊開始就可以看到你的 Chat ID。

### 第三步：部署到 Cloudflare Workers

直接來到 [Cloudflare Workers](https://workers.cloudflare.com/)，建立一個新的 Worker。

將 [telegram-notification-mcp](https://github.com/kstonekuan/telegram-notification-mcp) 的程式碼複製到你的 Worker 中。

然後在「設定」→「變數和祕密」中新增這兩個環境變數：
- `BOT_TOKEN`：你的 Telegram Bot Token
- `DEFAULT_CHAT_ID`：你的 Chat ID

完成後點擊部署就完成了！

### 第四步：讓 Claude Code 連接到你的通知服務

部署完成後，你會得到一個 Worker 的網址（類似 `https://your-worker-name.workers.dev/`）。現在需要告訴 Claude Code 怎麼連接到這個服務。

在終端機中執行以下命令：

```bash
# 添加 MCP 連線
claude mcp add telegram-notify https://your-worker-name.workers.dev/sse -t sse
```

記得把 `your-worker-name` 改成你實際的 Worker 名稱。

## 設定 Claude Code 通知

現在最重要的一步來了！需要在你的 `claude.md` 檔案中告訴 Claude Code 什麼時候要傳訊息。

設定是這樣的，可以參考：

```markdown
# Telegram 通知

使用 `mcp__telegram-notify__send_telegram_message` 工具向 Telegram 發送通知。

## 通知觸發時機

- **任務完成**：任務完全完成時必須發送通知
- **需要用戶輸入**：需要用戶輸入以繼續時必須發送通知
- **錯誤發生**：發生需要用戶注意的錯誤時必須發送通知
- **明確要求**：用戶明確要求通知時（例如：「通知我」、「傳訊息給我」、「讓我知道」）
- **重要里程碑**：專案達成重要里程碑時發送通知
- **長時間任務**：執行時間超過 5 分鐘的任務完成時通知

## 通知內容指南

### 基本原則
- 使用簡潔、資訊豐富的訊息
- 包含具體的狀態和數據
- 提供足夠的上下文資訊
- 使用表情符號提升可讀性

### 詳細內容建議

**建置/測試通知**
- 成功/失敗狀態和執行時間
- 測試通過/失敗的具體數量
- 錯誤類型和影響範圍

**錯誤通知**
- 具體錯誤訊息和發生位置
- 錯誤的嚴重程度
- 建議的解決方案

**進度通知**
- 當前完成的步驟
- 剩餘任務預估時間
- 可能的阻塞點

## 訊息格式範例

### 成功通知
```
✅ 建置完成 (2分34秒)
📊 測試結果：52/52 通過
🚀 部署成功至 staging 環境
```

### 錯誤通知
```
❌ 測試失敗：auth.test.ts
🔍 3/52 個測試失敗
📍 主要錯誤：JWT 驗證失敗
```

### 進度通知
```
🔄 正在處理中...
📈 進度：步驟 3/5 完成
⏱️ 預估剩餘時間：8分鐘
```

### 權限/輸入通知
```
⚠️ 需要權限修改 /etc/hosts
🔐 請確認是否繼續執行
```

### 安裝/更新通知
```
📦 套件安裝完成
➕ 新增：5 個套件
🔄 更新：3 個套件
```

## 高級功能

### 自訂訊息格式
```typescript
// 支援 Markdown 格式
{
  "text": "*重要*：建置失敗\n\n`錯誤代碼：E001`",
  "parse_mode": "Markdown"
}

// 支援 HTML 格式
{
  "text": "<b>部署完成</b>\n\n<code>環境：production</code>",
  "parse_mode": "HTML"
}
```

### 條件式通知
- 只在工作時間發送非緊急通知
- 根據錯誤嚴重程度調整通知頻率
- 支援靜音模式設定
```

## 實際使用感受

設定完成後，你就可以放心的讓 Claude Code 去跑任務了。不管是建置專案、跑測試、修復錯誤，完成的時候都會收到類似這樣的訊息：

```
✅ 建置完成 (2分34秒)
📊 測試結果：52/52 通過
🚀 部署成功至 staging 環境
```

如果有錯誤的話，也會收到具體的錯誤資訊：

```
❌ 測試失敗：auth.test.ts
🔍 3/52 個測試失敗
📍 主要錯誤：JWT 驗證失敗
```
```

這樣一來，你就可以安心的去吃飯、喝咖啡、或是處理其他事情，不用再一直盯著終端機了！

## 幾個使用小訣窿

### 訊息設計
建議訊息保持簡潔，但要有足夠的資訊。比如「建置完成」就比「好了」來得有用多了。

### 安全性
記得絕對不要在通知中包含任何敏感資訊，像是 API 金鑰、密碼等等。就算是錯誤訊息也要小心別洩漏這些。

### 監控使用量
偶爾去 Cloudflare Workers 的控制台看一下使用量，一般來說免費額度都很多了，但還是注意一下比較好。

## 如果沒有收到訊息怎麼辦？

### 首先檢查這幾個地方

**BOT_TOKEN 設定錯誤**
先確認一下你的 Token 有沒有設對，可以用這個指令測試：
```bash
curl -X GET "https://api.telegram.org/bot<你的BOT_TOKEN>/getMe"
```

**Chat ID 錯誤**
可以用這個指令測試看看能不能發送訊息：
```bash
curl -X POST "https://api.telegram.org/bot<你的BOT_TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "<你的CHAT_ID>", "text": "測試訊息"}'
```

**Cloudflare Workers 問題**
到 Cloudflare Workers 控制台看一下有沒有錯誤訊息，通常在「即時日誌」那裡可以找到。

### 其他可能的原因

- Bot 被你封鎖了（在聊天記錄中可以解除封鎖）
- Claude Code 沒有正確設定 MCP 連線
- 環境變數沒有儲存成功

## 結論

老實說，這個小功能真的大幅改善了我使用 Claude Code 的體驗。以前總是要一直盯著終端機，現在可以放心的去做其他事情，任務完成會自動通知，不用再擔心忘記了。

設定這個功能大概只要 10 分鐘，但可以用很久。如果你也有同樣的困擾，強烈建議你試試看！

希望這篇文章對你有幫助。如果有任何問題或建議，歡迎留言討論！