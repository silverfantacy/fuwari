---
title: 在 Ollama 中設置跨域資源共享 (CORS) 方法
published: 2024-04-26
description: Ollama 遇到跨域資源共享 (CORS) 問題時，需要在 macOS 和 Linux 上設置環境變數。
image: 'https://media.virtualxnews.com/2024/04/613be9f753a74ab8d87ca9e572b84a05.jpg'
tags: [Ollama, CORS]
category: AI
draft: false
---

使用 Ollama 架設 LLM 模型，如果搭配 GUI 介面（例如：[Open WebUI](https://github.com/open-webui/open-webui)、[Lobe Chat](https://github.com/lobehub/lobe-chat)）一起使用的話非常方便。

當 Ollama 和 GUI 介面串接時，可能會遇到跨域資源共享 (CORS) 的問題。

![Lobe Chat 遇到 CORS 問題](https://media.virtualxnews.com/2024/04/470f022d1117e8fcb5051ffd5f9024ef.png)

要解決 CORS 問題需要設置環境變數。根據使用的作業系統不同，設置環境變數的方法也有所不同。以下是在不同作業系統上設置環境變數的步驟：

## macOS

在 macOS 上，如果你將 Ollama 作為應用程式運行，設置環境變數可以使用 `launchctl` 命令。這種方法允許你定義哪些來源可以訪問你的資源。以下是設置環境變數的步驟：

### 允許所有域名

如果要允許所有域名請求你的應用程式資源，請使用以下命令：

```
launchctl setenv Ollama_ORIGINS "*"
```

### 允許特定域名

如果你想限制只有特定域名可以訪問，可以按照以下方式指定：

```
launchctl setenv Ollama_ORIGINS "google.com,linkedin.com"
```

設置完所需的環境變數後，重新啟動 Ollama 應用程式就可以了。

![Lobe Chat API 檢查通過](https://media.virtualxnews.com/2024/04/5cea754de8e0b3d0ef4abe7aedbad531.png)

![Lobe Chat 成功使用 phi3](https://media.virtualxnews.com/2024/04/234351faf063a6c232470ac88d5265ae.png)


## Linux（目前尚未解決 CORS 問題）

對於在 Linux 上執行 Ollama 作為 systemd 服務的用戶，可以使用 `systemctl` 命令設置環境變數。以下是設置環境變數的步驟：

### 允許所有域名

如果要允許所有域名請求你的應用程式資源，可以在服務文件中添加以下內容：

使用以下命令打開服務文件：

```
vim /etc/systemd/system/ollama.service
```

在 `[Service]` 部分，添加以下行以設置 CORS 設定：

```
Environment="Ollama_ORIGINS=*"
```

### 允許特定域名

如果你想限制只有特定域名可以訪問，可以按照以下方式指定：

```
Environment="Ollama_ORIGINS=google.com,linkedin.com"
```

保存更改後，重新載入 systemd 並重新啟動 Ollama：

```
systemctl daemon-reload
systemctl restart Ollama
```

雖然目前查到的資料是這樣，但是我在 Linux 上測試時，發現這樣設置並不能解決 CORS 問題。如果你有更好的解決方法，歡迎聯絡我。

> 參考資料：
> 
> [How to Handle CORS Settings in OLLAMA: A Comprehensive Guide](https://medium.com/dcoderai/how-to-handle-cors-settings-in-ollama-a-comprehensive-guide-ee2a5a1beef0)
> 
> [Ollama on Linux](https://github.com/ollama/ollama/blob/main/docs/linux.md)