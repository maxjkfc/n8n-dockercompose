# n8n 工作流自動化平台

這個專案使用 Docker Compose 來部署 n8n 工作流自動化平台及其所需的 PostgreSQL 資料庫。

## 系統需求

- Docker 與 Docker Compose
- 可用的 5678 和 5432 端口

## 快速開始

### 啟動服務

```bash
docker compose up -d
```

啟動後，服務將在背景運行。

### 檢查服務狀態

```bash
docker compose ps
```

### 停止服務

```bash
docker compose down
```

如果你想同時刪除所有數據卷（這將刪除所有工作流和設置）：

```bash
docker compose down -v
```

## 服務訪問

- **n8n 平台**: http://localhost:5678
- **登入憑證**: 
  - 用戶名: `admin`
  - 密碼: `admin`

## 服務配置

### n8n 服務

- 使用官方 n8nio/n8n 映像
- 配置使用 PostgreSQL 作為資料庫
- 啟用基本身份驗證
- 數據持久化存儲在 Docker 卷中

### PostgreSQL 資料庫

- 版本: PostgreSQL 13
- 資料庫名稱: `n8n`
- 用戶名: `n8n`
- 密碼: `n8n`
- 端口: 5432 (本機可訪問)
- 數據持久化存儲在 Docker 卷中

## 數據持久化

所有數據都存儲在 Docker 卷中，即使容器被刪除，數據也會保留：

- `n8n_data`: 存儲 n8n 配置和工作流
- `postgres_data`: 存儲 PostgreSQL 資料庫數據

## 自定義配置

如需修改配置，請編輯 `docker-compose.yml` 文件中的環境變數。常見的修改包括：

- 更改管理員密碼 (`N8N_BASIC_AUTH_PASSWORD`)
- 更改資料庫密碼 (`DB_POSTGRESDB_PASSWORD` 和 `POSTGRES_PASSWORD`)
- 修改 webhook URL (`WEBHOOK_URL`)

修改後需要重新啟動服務：

```bash
docker compose down
docker compose up -d
```

## 故障排除

### 無法連接到 n8n 界面

確認服務是否正在運行：

```bash
docker compose ps
```

檢查 n8n 容器日誌：

```bash
docker compose logs n8n
```

### 資料庫連接問題

檢查 PostgreSQL 容器是否正常運行：

```bash
docker compose ps postgres
```

查看 PostgreSQL 日誌：

```bash
docker compose logs postgres
```

## 備份與還原

### 備份

```bash
# 備份 n8n 工作流和設置
docker run --rm -v n8n_n8n_data:/source -v $(pwd):/backup alpine tar -czf /backup/n8n_data_backup.tar.gz -C /source .

# 備份 PostgreSQL 資料庫
docker compose exec postgres pg_dump -U n8n n8n > n8n_db_backup.sql
```

### 還原

```bash
# 還原 n8n 工作流和設置
docker run --rm -v n8n_n8n_data:/target -v $(pwd):/backup alpine sh -c "rm -rf /target/* && tar -xzf /backup/n8n_data_backup.tar.gz -C /target"

# 還原 PostgreSQL 資料庫
cat n8n_db_backup.sql | docker compose exec -T postgres psql -U n8n n8n
```
