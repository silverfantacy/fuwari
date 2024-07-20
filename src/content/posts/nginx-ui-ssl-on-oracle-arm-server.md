---
title: Oracle Arm 架構伺服器上使用 Nginx UI 與 SSL 憑證
published: 2024-07-20
description: 'Oracle Arm 架構伺服器上安裝 Nginx，並使用 Nginx UI 進行管理。以及如何使用 Certbot 取得與安裝 SSL 憑證'
image: 'https://media.virtualxnews.com/2024/07/b5f0f7d32ad4452c6707bf395a9f0745.png'
tags: [Nginx, Nginx UI, Oracle, Arm, SSL, Certbot]
category: '伺服器'
draft: false 
---

由於一次手癢的更新伺服器，不幸讓伺服器「翻車」了。距離上次重建伺服器已經有一段時間，這次藉此機會重新設定，並將過程記錄下來。
剛好之前發現了一個實用的工具 Nginx UI，它提供圖形化介面來管理 Nginx，甚至能自動更新 SSL 憑證。就裝來試試看，雖然目前 SSL 設定上還遇到一些問題，但會先記錄目前為止的步驟，待問題解決後再進行更新。

## 開始

建立一台 Oracle Arm 架構伺服器，並具備基本的 Linux 操作知識。

![Oracle Arm 架構伺服器](https://media.virtualxnews.com/2024/07/9dd872dbae4a7332cb2e2f73f97e5728.png)

**建議：** 在開始前，確保您的伺服器系統為最新狀態，並已設定好域名解析。

### 設定 root 密碼

1. 使用 SSH 登入伺服器。
2. 執行 `sudo su` 切換至 root 使用者。
3. 執行 `passwd` 並依照指示設定 root 密碼。

### 安裝 Nginx

**建議：** 參考 DigitalOcean 的 Nginx 安裝教學：[https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04)

```bash
sudo apt update
sudo apt install nginx
```

### 設定防火牆

由於 Oracle Linux 預設使用 iptables，因此我們需要調整防火牆設定：

1. 允許 ufw 規則
    
    ```bash
    sudo ufw allow 'Nginx HTTP'
    ```
    
2. 開啟 ufw
    
    ```bash
    sudo ufw enable
    ```
    
3. 要刪除 iptables 並重裝 ufw [(因為是Oracle的關係)](https://hiraku.dev/2020/04/6171/)
    
    ```bash
    apt-get remove iptables
    apt-get install ufw
    ```
    
4. 重開機，輸入檢查 ufw 是不是被自動關掉，是的話再打開一次，再重開機，確定重開機不會被關掉
    
    ```bash
    sudo ufw status
    sudo ufw enable
    ```

### 檢查 Nginx 狀態

```bash
sudo systemctl status nginx
```

### 安裝 Nginx UI

![Nginx UI](https://media.virtualxnews.com/2024/07/e48a97a2d0af6fbf5a1adf21772e0cab.png)

[Nginx UI](https://nginxui.com/zh_CN/guide/install-script-linux.html) 提供一個方便的圖形化介面來管理 Nginx 設定。

```bash
bash <(curl -L -s [https://mirror.ghproxy.com/https://raw.githubusercontent.com/0xJacky/nginx-ui/master/install.sh](https://mirror.ghproxy.com/https://raw.githubusercontent.com/0xJacky/nginx-ui/master/install.sh)) install -r [https://mirror.ghproxy.com/](https://mirror.ghproxy.com/)
```

**注意：** Nginx UI 預設監聽端口為 9000，首次開啟需註冊帳號密碼。

### 安裝 SSL 憑證 (需要域名)

![Certbot](https://media.virtualxnews.com/2024/07/f918495c1da7552267bd98485113932a.png)

我們將使用 [Certbot](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal) 自動取得與安裝 Let's Encrypt 的免費 SSL 憑證。

**建議：** 確保您的域名解析已正確設定，指向您的伺服器 IP。

:::note
如果 Domain是 gandi 的話，為了使用 Certbot 要先安裝 [python3-certbot-nginx](https://installati.one/install-python3-certbot-nginx-ubuntu-22-04/#install-python3-certbot-nginx-using-apt-get)
```bash
sudo apt-get update
sudo apt-get -y install python3-certbot-nginx
```
:::


1. **安裝 Certbot:**

```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

2. **取得並安裝憑證:**

```bash
sudo certbot --nginx
```

依照指示輸入您的域名，Certbot 將自動完成憑證的取得與安裝。

3. **測試自動續訂:**

```bash
sudo certbot renew --dry-run
```

### 開始使用 Nginx UI

1. 開啟瀏覽器，輸入 `http://<您的伺服器IP>:9000`。
2. 使用您註冊的帳號密碼登入。
3. 在 Nginx UI 中新增您的網站，並進行相關設定。
![Nginx UI 新增網站](https://media.virtualxnews.com/2024/07/b376f3e168668d1d2c11a3c1a8197050.png)
4. 需要進 server 重新執行，新設定的網域才有辦法加上 SSL
```bash
sudo certbot renew --dry-run
```

## 完成

已經成功在 Oracle Arm 架構伺服器上安裝 Nginx，並使用 Nginx UI 進行管理與安裝 SSL 憑證。

> 參考資料：
> 
> [Oracle Cloud Ubuntu 不用預設 iptables 的方法](https://hiraku.dev/2020/04/6171/)
> 
> [Nginx UI](https://nginxui.com/zh_CN/)
> 
> [Certbot](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)
> 
> [How To Install python3-certbot-nginx on Ubuntu 22.04](https://installati.one/install-python3-certbot-nginx-ubuntu-22-04/#install-python3-certbot-nginx-using-apt-get)