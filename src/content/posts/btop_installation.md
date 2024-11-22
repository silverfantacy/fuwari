---
title: 'btop：現代化的系統資源監控工具'
published: 2024-11-22
description: '介紹 btop 的安裝步驟、功能特點及設定參數。'
image: 'https://imager.virtualxnews.com/rest/WdhwUKK.png'
tags: [btop, 系統監控, Server]
category: '工具'
draft: false
---

### btop：現代化的系統資源監控工具

#### 什麼是 `btop`？

`btop` 是一款功能強大的系統資源監控工具，它是 `htop` 和 `bpytop` 的 C++ 重寫版本。透過精心設計的 TUI（文字使用者介面），它能即時顯示 CPU、記憶體、磁碟、網路和處理程序的詳細資訊，讓系統監控變得更直觀且高效。

#### 為什麼選擇 `btop`？

- **效能優異**：使用 C++ 編寫，即使在高負載下也能保持流暢運行
- **介面美觀**：支援真彩色顯示，提供清晰的資源使用視覺化圖表
- **高度客製化**：豐富的設定選項，可依需求調整顯示內容和外觀
- **跨平台支援**：支援 Linux、macOS 和 FreeBSD 等多種作業系統

#### 安裝方法

##### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install btop
```

##### macOS
```bash
brew install btop
```

#### 基本操作指南

1. **啟動程式**：
   ```bash
   btop
   ```

2. **常用快捷鍵**：
   - `h`：顯示說明
   - `m`：切換記憶體視圖
   - `1-4`：切換 CPU 核心顯示模式
   - `f`：開啟處理程序搜尋
   - `k`：終止選定的處理程序
   - `q`：退出程式

#### 重要設定參數

btop 提供了豐富的設定選項，可以通過編輯 `~/.config/btop/btop.conf` 或在程式內按 `Esc` 進行設定：

1. **顯示設定**
   - `color_theme`：設定顯示主題
   - `vim_keys`：啟用/停用 vim 式快捷鍵
   - `show_cpu_freq`：顯示 CPU 頻率
   - `update_ms`：更新間隔（毫秒）

2. **CPU 視圖設定**
   - `cpu_graph_upper`：CPU 圖表上半部顯示類型
   - `cpu_graph_lower`：CPU 圖表下半部顯示類型
   - `cpu_single_graph`：是否合併顯示所有 CPU 核心

3. **記憶體視圖設定**
   - `mem_graphs`：是否顯示記憶體使用圖表
   - `show_swap`：是否顯示 swap 使用情況
   - `swap_disk`：是否將 swap 顯示為磁碟

4. **處理程序視圖設定**
   - `proc_sorting`：處理程序排序方式
   - `proc_reversed`：是否反向排序
   - `proc_tree`：是否使用樹狀顯示
   - `tree_depth`：樹狀顯示深度

#### 進階使用技巧

1. **自訂主題**：
   - 可在 `~/.config/btop/themes` 建立自訂主題檔案
   - 使用 JSON 格式定義顏色方案

2. **效能優化**：
   - 調整更新間隔（update_ms）以平衡效能
   - 關閉不需要的圖表來減少系統負載

3. **處理程序管理**：
   - 使用處理程序搜尋快速定位特定程式
   - 支援多種信號方式終止處理程序

> 參考資料：
> 
> [btop GitHub Repository](https://github.com/aristocratos/btop)
> [btop 官方文件](https://github.com/aristocratos/btop/blob/main/README.md)