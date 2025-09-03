# PM-AI 快速開始指南

## 🚀 5 分鐘快速啟動

### 1. 環境準備
```bash
# 複製專案
git clone <repository-url>
cd pm-ai

# 複製環境變數
cp env.example .env

# 啟動開發環境
docker-compose -f infra/docker/docker-compose.dev.yml up -d
```

### 2. 驗證服務
```bash
# 檢查服務狀態
docker-compose -f infra/docker/docker-compose.dev.yml ps

# 測試 API
curl http://localhost:8000/health

# 測試前端
curl http://localhost:3000
```

### 3. 初始化資料
```bash
# 初始化資料庫
docker-compose -f infra/docker/docker-compose.dev.yml exec api python -m alembic upgrade head

# 載入初始資料
docker-compose -f infra/docker/docker-compose.dev.yml exec api python scripts/load_seed_data.py
```

## 📋 服務清單

| 服務 | 網址 | 說明 |
|------|------|------|
| 前端應用 | http://localhost:3000 | Next.js 護理師工作台 |
| API 服務 | http://localhost:8000 | FastAPI 後端服務 |
| API 文件 | http://localhost:8000/docs | Swagger UI |
| 監控面板 | http://localhost:3001 | Grafana 儀表板 |
| 向量資料庫 | http://localhost:6333 | Qdrant 管理介面 |
| 物件儲存 | http://localhost:9001 | MinIO 管理介面 |

## 🛠️ 開發工具

### Cursor 內建 Prompt 使用
1. 開啟 `docs/prompts/` 中的任一 Prompt 檔案
2. 複製內容到 Cursor 的內建 Prompt 設定
3. 開始使用自動化工作流程

### 常用命令
```bash
# 查看日誌
docker-compose -f infra/docker/docker-compose.dev.yml logs -f api

# 重啟服務
docker-compose -f infra/docker/docker-compose.dev.yml restart api

# 進入容器
docker-compose -f infra/docker/docker-compose.dev.yml exec api bash

# 清理環境
docker-compose -f infra/docker/docker-compose.dev.yml down -v
```

## 🔧 故障排除

### 常見問題
1. **端口衝突**：檢查 3000, 8000, 5432, 6333, 9000 端口是否被占用
2. **資料庫連線失敗**：等待 PostgreSQL 完全啟動後再啟動 API 服務
3. **向量資料庫錯誤**：檢查 Qdrant 服務狀態和磁碟空間

### 日誌查看
```bash
# 查看所有服務日誌
docker-compose -f infra/docker/docker-compose.dev.yml logs

# 查看特定服務日誌
docker-compose -f infra/docker/docker-compose.dev.yml logs api
docker-compose -f infra/docker/docker-compose.dev.yml logs web
```

## 📚 下一步

1. 閱讀 [README.md](README.md) 了解完整系統架構
2. 查看 [OpenAPI 文件](docs/openapi/openapi.yml) 了解 API 規格
3. 使用 [Cursor Prompt](docs/prompts/) 開始敏捷開發
4. 參考 [Docker 配置](infra/docker/) 進行部署

## 🆘 需要幫助？

- 查看 [README.md](README.md) 的詳細說明
- 檢查 [OpenAPI 文件](docs/openapi/openapi.yml) 的 API 規格
- 使用 [Cursor Prompt](docs/prompts/) 進行自動化工作流程
- 聯繫開發團隊：dev@pm-ai.com
