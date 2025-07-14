---
title: Claude Code MCP 設定研究：跨專案配置經驗分享
published: 2025-07-14
description: '深入探討 Claude Code MCP 的三種作用域設定差別，以及如何實現跨專案和跨電腦的統一配置，解決開發者在多個環境下的 MCP 工具管理需求。'
image: ''
tags: [Claude Code, MCP, AI, 開發工具, 設定管理]
category: 'AI'
draft: false
---

## 前言

在多台電腦上都安裝了 Claude Code，並希望在所有專案中都能使用相同的 MCP（Model Context Protocol）伺服器時，理解不同作用域的配置方式就變得至關重要。本文會探討 Claude Code MCP 的三種作用域設定，以及如何實現跨專案的統一配置。

## MCP 作用域概述

Claude Code 提供三種 MCP 伺服器作用域，每種都有其特定的使用場景和配置方式：

### 1. 本地作用域（Local Scope）

**特性**：
- 預設配置方式
- 用戶私有，僅在當前專案中可用
- 適合存放敏感憑證和實驗性配置

**配置檔案位置**：
```
~/.claude.json
```

**使用場景**：
- 個人開發環境的特殊配置
- 包含敏感 API 金鑰的伺服器
- 實驗性或測試用的 MCP 伺服器

**設定指令**：
```bash
claude mcp add my-private-server /path/to/server
```

### 2. 用戶作用域（User Scope）

**特性**：
- 用戶私有，但跨所有專案可用
- 解決跨專案 MCP 工具共用需求
- 只影響當前用戶帳號

**配置檔案位置**：
```
~/.claude.json
```

**使用場景**：
- 個人常用的工具和 API 服務
- 需要在多個專案間復用的 MCP 伺服器
- 不需要與團隊成員共享的個人工具

**設定指令**：
```bash
claude mcp add my-user-server -s user /path/to/server
```

### 3. 專案作用域（Project Scope）

**特性**：
- 可被專案團隊所有成員共享
- 通過版本控制系統同步
- 需要用戶確認才能使用

**配置檔案位置**：
```
<專案根目錄>/.mcp.json
```

**使用場景**：
- 團隊協作必需的外部服務
- 專案特定的 MCP 工具
- 需要確保團隊一致性的配置

**設定指令**：
```bash
claude mcp add shared-server -s project /path/to/server
```

## 作用域差別詳細說明

根據官方文件和實際使用經驗，以下是三種作用域的詳細比較：

| 作用域      | 可訪問性           | 配置存放位置         | 適用場景                | 是否可團隊共享 | 優先級 |
| ----------- | ---------------- | ------------------- | ----------------------- | -------------- | ------ |
| **local**   | 當前用戶、當前專案 | `~/.claude.json`    | 私有、實驗、敏感憑證    | 否             | 最高   |
| **user**    | 當前用戶、所有專案 | `~/.claude.json`    | 復用個人常用服務器      | 否             | 最低   |
| **project** | 所有項目成員      | 項目根目錄`.mcp.json`| 團隊協作、服務共用      | 是             | 中等   |

### 作用域優先級

當同名 MCP 伺服器存在於多個作用域時，Claude Code 會按照以下優先級順序載入：

1. **本地作用域** (Local) - 最高優先級
2. **專案作用域** (Project) - 中等優先級  
3. **用戶作用域** (User) - 最低優先級

這種設計確保了個人配置可以覆蓋團隊配置，而專案配置可以覆蓋全域配置。

## 跨專案 MCP 設定的最佳解決方案

針對「在不同電腦上都有裝 Claude Code，希望設定一次 MCP 所有專案都可以使用」的需求，**用戶作用域（User Scope）**是最佳選擇。

### 實作步驟

#### 1. 設定用戶作用域 MCP 伺服器

以 Brave Search 為例：

```bash
claude mcp add brave-search --scope user \
  --env BRAVE_API_KEY=your_api_key_here \
  -- npx -y @modelcontextprotocol/server-brave-search
```

#### 2. 設定多個常用 MCP 工具

```bash
# Perplexity Search
claude mcp add perplexity --scope user \
  --env PERPLEXITY_API_KEY=your_key \
  -- npx -y @modelcontextprotocol/server-perplexity

# File System Access
claude mcp add filesystem --scope user \
  -- npx -y @modelcontextprotocol/server-filesystem

# Git Integration
claude mcp add git --scope user \
  -- npx -y @modelcontextprotocol/server-git
```

#### 3. 驗證設定

```bash
# 列出所有 MCP 伺服器
claude mcp list

# 檢查特定伺服器詳細資訊
claude mcp get brave-search
```

### 配置檔案結構

用戶作用域的配置會寫入 `~/.claude.json`，結構如下：

```json
{
  "mcpServers": {
    "brave-search": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your_api_key_here"
      }
    },
    "filesystem": {
      "type": "stdio", 
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"]
    }
  }
}
```

## 多台電腦同步策略

### 方法一：手動同步配置檔案

將 `~/.claude.json` 複製到其他電腦的相同位置：

```bash
# 從主電腦複製
scp ~/.claude.json user@target-machine:~/

# 或使用雲端同步
cp ~/.claude.json ~/Dropbox/claude-settings/
```

### 方法二：使用版本控制

將 Claude 設定納入個人的 dotfiles 管理：

```bash
# 建立 dotfiles 倉庫
mkdir ~/dotfiles
cd ~/dotfiles
git init

# 加入 Claude 設定
ln -s ~/.claude.json claude.json
git add claude.json
git commit -m "Add Claude Code MCP settings"

# 在其他電腦上
git clone your-dotfiles-repo ~/dotfiles
ln -s ~/dotfiles/claude.json ~/.claude.json
```

### 方法三：環境變數管理

對於包含敏感資訊的 MCP 設定，建議使用環境變數：

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中設定
export BRAVE_API_KEY="your_api_key"
export PERPLEXITY_API_KEY="your_perplexity_key"

# MCP 設定中引用環境變數
claude mcp add brave-search --scope user \
  --env BRAVE_API_KEY=$BRAVE_API_KEY \
  -- npx -y @modelcontextprotocol/server-brave-search
```

## 設定檔案管理最佳實踐

### 1. 分層配置策略

- **用戶作用域**：個人常用工具（如搜尋引擎、檔案系統）
- **專案作用域**：專案特定工具（如特定 API、專案工具）
- **本地作用域**：敏感或實驗性配置

### 2. 安全性考量

```bash
# 確保配置檔案權限安全
chmod 600 ~/.claude.json

# 避免在專案作用域中存放敏感資訊
# 使用環境變數或本地作用域替代
```

### 3. 版本控制注意事項

```gitignore
# 在 .gitignore 中排除敏感配置
.claude/settings.local.json
.env.local
```

## 常見問題與解決方案

### Q1: 如何檢視目前所有 MCP 設定？

```bash
claude mcp list
```

### Q2: 如何移除不需要的 MCP 伺服器？

```bash
claude mcp remove server-name
```

### Q3: 如何更新 MCP 伺服器配置？

```bash
# 先移除舊配置
claude mcp remove server-name

# 再重新加入新配置
claude mcp add server-name --scope user new-command
```

### Q4: 配置在不同專案中不生效？

檢查作用域設定和優先級：

```bash
# 確認伺服器作用域
claude mcp get server-name

# 檢查是否被其他作用域覆蓋
claude mcp list --scope all
```

### Q5: 環境變數設定問題

確保環境變數在所有 shell 環境中都可用：

```bash
# 檢查環境變數
echo $BRAVE_API_KEY

# 永久設定（加入 ~/.bashrc 或 ~/.zshrc）
echo 'export BRAVE_API_KEY="your_key"' >> ~/.bashrc
source ~/.bashrc
```

## 推薦的 MCP 工具配置

### 基礎工具套件

```bash
# 檔案系統操作
claude mcp add filesystem --scope user \
  -- npx -y @modelcontextprotocol/server-filesystem

# Git 版本控制
claude mcp add git --scope user \
  -- npx -y @modelcontextprotocol/server-git

# 搜尋引擎
claude mcp add brave-search --scope user \
  --env BRAVE_API_KEY=$BRAVE_API_KEY \
  -- npx -y @modelcontextprotocol/server-brave-search
```

### 進階工具套件

```bash
# Perplexity AI 搜尋
claude mcp add perplexity --scope user \
  --env PERPLEXITY_API_KEY=$PERPLEXITY_API_KEY \
  -- npx -y @modelcontextprotocol/server-perplexity

# SQLite 資料庫操作
claude mcp add sqlite --scope user \
  -- npx -y @modelcontextprotocol/server-sqlite

# 時間和行事曆
claude mcp add time --scope user \
  -- npx -y @modelcontextprotocol/server-time
```

## 總結

對於跨專案和跨電腦的 MCP 設定需求，**用戶作用域**提供了最佳的平衡點：

1. **便利性**：一次設定，所有專案可用
2. **隱私性**：不會洩露給其他用戶或團隊
3. **可攜性**：易於在多台電腦間同步

通過合理運用三種作用域，可以建立一個既靈活又安全的 MCP 設定體系，在提升開發效率的同時，確保配置的一致性和安全性。

建議從設定幾個常用的用戶作用域 MCP 伺服器開始，例如搜尋工具、檔案系統存取等，然後根據實際需求逐步完善您的 MCP 工具鏈。

### 下一步行動

1. 安裝並設定基礎 MCP 工具套件
2. 設定環境變數管理敏感資訊
3. 建立個人的 dotfiles 倉庫進行同步
4. 根據專案需求逐步擴充 MCP 工具

透過本文的分享，應該能夠建立一套適合自己工作流程的 Claude Code MCP 配置系統，大幅提升 AI 輔助開發的效率。