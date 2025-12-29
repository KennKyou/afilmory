# Zeabur 部署指南

## 部署架構

這個專案在 Zeabur 上的部署包含以下服務：

1. **Core 後端服務** - NestJS API（端口 1841）
2. **SSR 前端服務** - Next.js 應用（端口 3000）
3. **PostgreSQL 資料庫** - Zeabur 預構建服務
4. **Redis 快取** - Zeabur 預構建服務

## 部署步驟

### 1. 創建 Zeabur 專案

1. 登入 [Zeabur](https://zeabur.com)
2. 點擊 "New Project" 創建新專案
3. 選擇適合的區域（推薦：Hong Kong 或 Tokyo）

### 2. 添加預構建服務

#### 添加 PostgreSQL

1. 在專案中點擊 "Add Service"
2. 選擇 "Prebuilt" → "PostgreSQL"
3. 等待服務啟動
4. Zeabur 會自動生成 `DATABASE_URL` 環境變數

#### 添加 Redis

1. 在專案中點擊 "Add Service"
2. 選擇 "Prebuilt" → "Redis"
3. 等待服務啟動
4. Zeabur 會自動生成 `REDIS_URL` 環境變數

### 3. 部署後端服務 (Core)

1. 點擊 "Add Service" → "Git"
2. 選擇你的 GitHub 倉庫
3. Zeabur 會自動檢測到 `zeabur.json` 並識別 `core` 服務
4. 設置以下環境變數：

```bash
# 必需環境變數
NODE_ENV=production
HOSTNAME=0.0.0.0
PORT=1841

# 資料庫連接（使用 Zeabur PostgreSQL 服務的自動注入變數）
DATABASE_URL=${POSTGRES_CONNECTION_STRING}

# Redis 連接（使用 Zeabur Redis 服務的自動注入變數）
REDIS_URL=${REDIS_CONNECTION_STRING}

# 加密密鑰（請更換為你自己的密鑰）
CONFIG_ENCRYPTION_KEY=your_secret_encryption_key_here

# S3 配置（如果使用）
S3_ACCESS_KEY_ID=your_s3_access_key
S3_SECRET_ACCESS_KEY=your_s3_secret_key

# Git Token（如果需要）
GIT_TOKEN=your_git_token
```

5. 點擊 "Deploy" 開始部署

### 4. 部署前端服務 (SSR)

1. 點擊 "Add Service" → "Git"
2. 選擇同一個 GitHub 倉庫
3. Zeabur 會識別到 `ssr` 服務
4. 設置以下環境變數：

```bash
# 必需環境變數
NODE_ENV=production

# API 端點（指向你的 Core 服務）
# Zeabur 會自動為每個服務生成域名，使用格式：
# https://<service-name>-<project-name>.zeabur.app
# 或使用內部網絡：core.zeabur.internal:1841
NEXT_PUBLIC_API_URL=https://core-<your-project-name>.zeabur.app

# 其他必要的環境變數...
```

5. 點擊 "Deploy" 開始部署

### 5. 配置域名（可選）

1. 在每個服務的設置中，點擊 "Domains"
2. 添加自定義域名
3. 按照提示配置 DNS 記錄

## 環境變數參考

### Core 服務需要的環境變數

```bash
# 基礎配置
NODE_ENV=production
HOSTNAME=0.0.0.0
PORT=1841

# 資料庫
DATABASE_URL=${POSTGRES_CONNECTION_STRING}

# Redis
REDIS_URL=${REDIS_CONNECTION_STRING}

# 安全
CONFIG_ENCRYPTION_KEY=<generate-a-secure-key>

# 存儲（根據你的配置）
S3_ACCESS_KEY_ID=<your-key>
S3_SECRET_ACCESS_KEY=<your-secret>
S3_BUCKET=<your-bucket>
S3_REGION=<your-region>
S3_ENDPOINT=<your-endpoint>

# 其他
GIT_TOKEN=<if-needed>
```

### SSR 服務需要的環境變數

查看你的 `apps/ssr` 目錄下的 `.env.example` 或相關配置文件。

## 服務間通信

在 Zeabur 上，服務可以通過以下方式互相通信：

1. **公開 URL**：`https://<service-name>-<project-name>.zeabur.app`
2. **內部網絡**：`<service-name>.zeabur.internal:<port>`（推薦，更快且安全）

例如，SSR 服務訪問 Core 服務：
- 公開：`https://core-myproject.zeabur.app`
- 內部：`http://core.zeabur.internal:1841`

## 資料庫遷移

首次部署後，你需要運行資料庫遷移：

1. 在 Core 服務的控制台中，找到 "Terminal" 或 "Logs"
2. 執行遷移命令（根據你的專案配置）

或者，你可以在本地執行：

```bash
# 設置環境變數指向 Zeabur 的資料庫
export DATABASE_URL="<zeabur-postgres-connection-string>"

# 執行遷移
pnpm --filter core db:migrate
```

## 監控和日誌

1. 在 Zeabur 控制台中，每個服務都有 "Logs" 標籤
2. 可以查看實時日誌和歷史日誌
3. 使用 "Metrics" 標籤查看 CPU、記憶體等使用情況

## 常見問題

### 1. 構建失敗

- 檢查 `zeabur.json` 配置是否正確
- 確保所有依賴都在 `package.json` 中正確聲明
- 查看構建日誌中的錯誤信息

### 2. 服務無法啟動

- 檢查環境變數是否正確設置
- 確保資料庫和 Redis 服務已經啟動
- 查看服務日誌

### 3. 無法連接資料庫

- 確認 `DATABASE_URL` 環境變數正確設置
- 在 Zeabur 中，使用 `${POSTGRES_CONNECTION_STRING}` 自動引用 PostgreSQL 服務
- 檢查資料庫服務是否健康

### 4. 構建時間過長

- Zeabur 的構建可能需要 5-10 分鐘，特別是首次部署
- 如果超過 15 分鐘，檢查構建日誌
- 考慮優化 Dockerfile 中的構建步驟

## 成本估算

Zeabur 的計費基於資源使用：

- 免費額度：適合測試和小型專案
- 按需付費：根據 CPU、記憶體、流量計費
- 預構建服務（PostgreSQL、Redis）有額外費用

建議在測試階段使用較小的資源配置。

## 自動部署

在 Zeabur 中連接 GitHub 倉庫後：

1. 每次推送到主分支會自動觸發部署
2. 可以在設置中配置自動部署的分支
3. 支持 Preview Deployments（PR 預覽）

## 需要幫助？

- [Zeabur 官方文檔](https://zeabur.com/docs)
- [Zeabur Discord 社群](https://discord.gg/zeabur)
- [專案 GitHub Issues](https://github.com/Afilmory/Afilmory/issues)
