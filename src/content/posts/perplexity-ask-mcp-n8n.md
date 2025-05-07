---
title: 如何在 n8n 中使用 Perplexity-Ask MCP 進行即時網路搜尋
description: 本文介紹如何在 n8n 自動化平台中整合 Perplexity-Ask MCP，實現 AI 即時網路搜尋，並提供安裝設定步驟、常見問題與參考連結。
published: 2025-05-07
image: 'https://imager.virtualxnews.com/rest/j2pTKSK.png'
tags: [AI平台, MCP, Perplexity, n8n, 自動化部署]
category: AI
draft: false
---

# 如何在 n8n 中使用 Perplexity-Ask MCP 進行即時網路搜尋

隨著 AI 與自動化工具的結合，n8n 也能透過 Perplexity-Ask MCP 實現即時網路搜尋，讓你的自動化流程具備最新的資訊查詢能力。本文將說明整合步驟、常見問題與參考資源。

## 什麼是 Perplexity-Ask MCP？

Perplexity-Ask MCP（Model Context Protocol）是一個開放標準，讓 AI 助手能即時存取網路資訊。透過 MCP server，n8n 等自動化平台可直接查詢 Perplexity 的 Sonar API，取得最新、最相關的網路內容。

- 官方說明：[Perplexity MCP Server 文件](https://docs.perplexity.ai/guides/mcp-server)
- GitHub Repo：[ppl-ai/modelcontextprotocol](https://github.com/ppl-ai/modelcontextprotocol/tree/main)

## 設定步驟

1. **申請 Perplexity API Key**
   - 前往 [Perplexity API 設定頁](https://www.perplexity.ai/settings/api) 申請金鑰。

2. **n8n 新增 MCP Client (STDIO) 帳號**
   - 在 n8n 的 Credentials 頁面新增 MCP Client (STDIO)。
   - `Command` 欄位填入：
     ```
     npx
     ```
   - `Arguments` 欄位填入：
     ```
     -y server-perplexity-ask
     ```
   - `Environments` 欄位填入：
     ```
     PERPLEXITY_API_KEY=你的API金鑰
     ```
   - 其他欄位依需求填寫。
   ![n8n 新增 MCP Client (STDIO) 帳號](https://imager.virtualxnews.com/rest/G8TtKSK.png)

3. **建立 workflow 串接 MCP 工具**
   - 於 workflow 中加入 MCP Client Tool 節點，選擇剛剛建立的帳號。
   - 可搭配 LangChain agent、xAI Grok 等模型，實現自動化 AI 查詢。

## AI Agent 串接 MCP 的正確流程

在 n8n 中，若要讓 AI Agent 能正確呼叫 MCP 工具（如 Perplexity），必須遵循以下步驟：

1. **System message 設定**
   - 建議在 AI Agent 的 system message 中加入：
     ```
     You are a helpful assistant
     - Use Perplexity for web search
     Make sure to listTools first then
     executeTool when ready
     ```
   - 這樣能引導 AI 先取得可用工具清單，再執行查詢。

2. **必須先 List Tools**
   - 在 workflow 中，需先建立一個 MCP Client Tool 節點，operation 設為 `listTools`。
   - 這步驟會回傳目前 MCP Server 支援的所有工具（如 perplexity_ask），以及每個工具可用的參數格式。

3. **設定 Execute Tool Name 與 Tool Parameters**
   - 取得工具清單後，AI Agent 會根據需求選擇正確的 Tool Name（如 `perplexity_ask`），並依照工具定義的參數格式組合 Tool Parameters。
   - 在 n8n 的 MCP Client Tool 節點，operation 設為 `executeTool`，Tool Name 可用：
     ```
     {{ $fromAI("tool","the select tool to use") }}
     ```
   - Tool Parameters 建議由 AI 自動帶入，或根據 listTools 回傳的格式手動設定。

4. **執行查詢**
   - 當 Tool Name 與 Tool Parameters 設定正確後，即可正確呼叫 MCP 工具，取得即時網路搜尋結果。

:::tip
一定要先 listTools，取得工具名稱與參數格式，才能正確設定 executeTool 的 Tool Name 與 Tool Parameters，否則會出現錯誤或查詢失敗。
:::

## 常見問題

- **MCP error -32000**
  - 請確認 `Command` 欄位為 `npx -y server-perplexity-ask`。
  - `PERPLEXITY_API_KEY` 是否正確。
  - 建議 n8n 版本升級至 1.91.0 以上。
  - n8n-nodes-mcp 版本升級至 0.1.23 以上。
  - 參考：[n8n 社群討論](https://community.n8n.io/t/mcp-perplexity/93711)

- **API Key 權限不足或格式錯誤**
  - 請確認 API Key 為 pplx-xxxx... 格式。

## 參考連結

- [Perplexity 官方 MCP Server 文件](https://docs.perplexity.ai/guides/mcp-server)
- [n8n 官方 Perplexity AI 範例](https://n8n.io/workflows/2824-query-perplexity-ai-from-your-n8n-workflows/)
- [ppl-ai/modelcontextprotocol GitHub](https://github.com/ppl-ai/modelcontextprotocol/tree/main)
- [n8n 社群討論 MCP + Perplexity](https://community.n8n.io/t/mcp-perplexity/93711)

---

如需 workflow JSON 範例或進階應用，歡迎留言討論！ 