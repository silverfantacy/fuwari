---
title: GitLab CI/CD 進階配置：環境變數與平行部署
published: 2024-11-14
description: '完整解析如何設定 GitLab CI/CD 的環境變數、共用腳本與平行部署，並提供實用的部署策略和範例。'
image: ''
tags: [GitLab, CICD, DevOps, 自動化部署]
category: '後端'
draft: false 
---

## 前言

在開發團隊中，一個穩定且高效的自動化部署流程是不可或缺的。GitLab CI/CD 提供了強大的功能來實現這個目標，但要妥善配置這些功能需要一些技巧。本文將從實務角度出發，詳細說明如何設定環境變數、撰寫部署腳本，以及實現多台伺服器的平行部署。

## CI/CD 配置基礎

讓我們先來看看一個典型的 `.gitlab-ci.yml` 檔案結構：

```yaml
# 全域變數設定
variables:
  PUBLISH_PATH: "./dist/"

# 預設設定
default:
  image: node:18.1.0-alpine
  tags: [docker]

# 定義部署階段
stages:
  - build
  - deploy
```

這是最基本的配置，接下來我們會逐步加入更多功能。

## 善用環境變數

在管理不同環境的部署配置時，善用環境變數可以讓整個流程更加靈活且易於維護。

### 使用錨點管理環境配置

YAML 的錨點（Anchors）功能讓我們能夠重複使用相同的配置：

```yaml
# 測試環境
.beta_vars: &beta_vars
  SFTP_USER: "deploy_user"
  SFTP_HOST: "beta.example.com"
  SFTP_PORT: "22"
  REMOTE_PATH: "/var/www/html/beta"

# 預發環境
.stage_vars: &stage_vars
  SFTP_USER: "deploy_user"
  SFTP_HOST: "stage.example.com"
  SFTP_PORT: "22"
  REMOTE_PATH: "/var/www/html/stage"

# 正式環境（多台伺服器）
.main_vars: &main_vars
  SFTP_USER: "deploy_user"
  SFTP_PORT: "22"
  REMOTE_PATH: "/var/www/html/prod"
  SFTP_HOST_1: "prod1.example.com"
  SFTP_HOST_2: "prod2.example.com"
```

## 部署工具介紹

在開始寫部署腳本之前，我們需要了解兩個重要的部署工具。

### sshpass：自動化 SSH 連線

sshpass 讓我們能夠在腳本中自動處理 SSH 密碼：

```bash
# 基本用法
sshpass -p "密碼" ssh user@host "指令"

# 在 CI/CD 中的使用方式
sshpass -p "${SFTP_PASSWORD}" ssh -p ${SFTP_PORT} ${SFTP_USER}@${SFTP_HOST} "指令"
```

> 注意：雖然 sshpass 很方便，但從安全性考量，建議在正式環境使用 SSH 金鑰認證。

### rsync：高效的檔案同步

rsync 是一個強大的檔案同步工具，特別適合用於部署：

```bash
# 常用參數說明
rsync -avz
# -a：保留檔案屬性
# -v：顯示進度
# -z：傳輸時壓縮
```

## 部署策略實作

我們採用了一個安全且高效的部署策略，主要分為三個步驟：

1. 建立臨時目錄
2. 同步檔案
3. 原子性切換

### 共用部署腳本

```yaml
.deploy-script: &deploy_script |
  # 1. 建立臨時目錄
  TEMP_PATH="${REMOTE_PATH}_temp"
  sshpass -p "${SFTP_PASSWORD}" ssh -o StrictHostKeyChecking=no \
    -p ${SFTP_PORT} ${SFTP_USER}@${SFTP_HOST} "mkdir -p '${TEMP_PATH}'"
  
  # 2. 同步檔案到臨時目錄
  sshpass -p "${SFTP_PASSWORD}" rsync -avz \
    -e "ssh -p ${SFTP_PORT}" \
    ${PUBLISH_PATH}/* \
    ${SFTP_USER}@${SFTP_HOST}:${TEMP_PATH}/
  
  # 3. 原子性切換目錄
  sshpass -p "${SFTP_PASSWORD}" ssh -p ${SFTP_PORT} \
    ${SFTP_USER}@${SFTP_HOST} "
    cd ${REMOTE_PATH%/*} &&
    rm -rf ${REMOTE_PATH##*/} &&
    mv ${TEMP_PATH} ${REMOTE_PATH##*/} &&
    chmod -R 755 ${REMOTE_PATH##*/}
  "
```

## 實際部署配置

### 單一環境部署

```yaml
deploy-beta:
  stage: deploy
  environment:
    name: beta
    url: https://beta.example.com
  variables:
    <<: *beta_vars
  only:
    - beta
  needs:
    - build-beta
  before_script:
    - apk add --update --no-cache openssh sshpass rsync
  script:
    - *deploy_script
```

### 多伺服器平行部署

```yaml
deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://example.com
  variables:
    <<: *main_vars
  only:
    - main
  parallel:
    matrix:
      - SFTP_HOST: ["$SFTP_HOST_1", "$SFTP_HOST_2"]
  script:
    - *deploy_script
```

## 安全性考量

在設定 CI/CD 時，安全性是重要的考量點：

1. **敏感資訊處理**
   - 使用 GitLab CI/CD 變數功能
   - 啟用 Protected 和 Masked 選項
   - 避免在配置檔中明文存放密碼

2. **存取控制**
   - 限制部署權限
   - 使用環境專用的部署帳號
   - 定期更新存取憑證

## 效能優化建議

1. **使用快取**
   ```yaml
   cache:
     paths:
       - node_modules/
     key: ${CI_COMMIT_REF_SLUG}
   ```

2. **最小化部署內容**
   - 只部署必要的檔案
   - 使用 `.gitignore` 和 `.dockerignore`
   - 善用 rsync 的排除功能

## 常見問題處理

1. **部署失敗檢查清單**
   - 網路連線狀態
   - 伺服器磁碟空間
   - 檔案權限設定
   - 環境變數值

2. **除錯技巧**
   - 使用 `set -x` 顯示詳細執行過程
   - 檢查 GitLab CI/CD 日誌
   - 驗證環境變數值：`echo $VARIABLE_NAME`

## 參考資料

- [GitLab CI/CD Variables](https://docs.gitlab.com/ee/ci/variables/)
- [GitLab CI/CD Parallel Jobs](https://docs.gitlab.com/ee/ci/yaml/#parallel)
- [GitLab CI/CD Best Practices](https://docs.gitlab.com/ee/ci/yaml/README.html#anchors)