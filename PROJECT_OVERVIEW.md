# PM-AI 專案總覽

## 📁 專案結構

```
pm-ai/
├── 📄 README.md                    # 主要系統文件
├── 📄 QUICKSTART.md                # 快速開始指南
├── 📄 PROJECT_OVERVIEW.md          # 專案總覽（本檔案）
├── 📄 env.example                  # 環境變數範本
│
├── 📁 apps/                        # 應用程式
│   ├── 📁 web/                     # Next.js 前端應用
│   └── 📁 api/                     # FastAPI 後端服務
│
├── 📁 services/                    # 微服務
│   ├── 📁 llm_gateway/             # LLM 代理服務
│   ├── 📁 rag_indexer/             # RAG 索引服務
│   └── 📁 kpi_daemon/              # KPI 監控服務
│
├── 📁 adapters/                    # 外部系統適配器
│   └── 📁 fhir/                    # FHIR/EMR 整合
│
├── 📁 data/                        # 資料相關
│   ├── 📁 seeds/                   # 初始資料
│   └── 📁 migrations/              # 資料庫遷移
│
├── 📁 infra/                       # 基礎設施
│   ├── 📁 docker/                  # Docker 配置
│   │   ├── 📄 docker-compose.dev.yml
│   │   ├── 📄 nginx.conf
│   │   └── 📄 prometheus.yml
│   └── 📁 k8s/                     # Kubernetes 部署
│
└── 📁 docs/                        # 文件
    ├── 📁 prd/                     # 產品需求文件
    ├── 📁 prompts/                 # Cursor 內建 Prompt
    │   ├── 📄 prompt_meeting_minutes.md
    │   ├── 📄 prompt_kpi_daemon.md
    │   ├── 📄 prompt_risk_matrix.md
    │   ├── 📄 prompt_change_control.md
    │   ├── 📄 prompt_mvp_acceptance.md
    │   └── 📄 prompt_backlog.md
    └── 📁 openapi/                 # API 規格
        └── 📄 openapi.yml
```

## 🎯 核心功能

### 1. QA 衛教機器人
- **功能**：糖腎共病衛教問答
- **技術**：RAG + LLM + 向量檢索
- **API**：`/api/qa/ask`

### 2. 對話記錄管理
- **功能**：護理對話記錄與追蹤
- **技術**：WebSocket + 事件溯源
- **API**：`/api/convo/*`

### 3. 報告生成
- **功能**：自動生成護理報告
- **技術**：LLM 結構化輸出 + PDF 生成
- **API**：`/api/report/*`

### 4. KPI 追蹤
- **功能**：專案指標監控與風險預警
- **技術**：定時任務 + 閾值監控
- **API**：`/api/kpi/*`

### 5. 變更控制
- **功能**：CAB 變更管理流程
- **技術**：工作流程引擎
- **API**：`/api/change/*`

## 🛠️ 技術架構

### 前端技術棧
- **框架**：Next.js 14 + React 18
- **樣式**：Tailwind CSS
- **狀態管理**：Zustand
- **表單**：React Hook Form
- **圖表**：Chart.js / Recharts

### 後端技術棧
- **框架**：FastAPI + Python 3.11
- **資料庫**：PostgreSQL 16
- **快取**：Redis 7
- **向量庫**：Qdrant
- **物件儲存**：MinIO / S3

### AI/ML 技術棧
- **LLM**：MedGemma / GPT-OSS 20B
- **嵌入模型**：sentence-transformers
- **RAG**：多路徑檢索 + 重排序
- **向量化**：OpenAI Embeddings

### 基礎設施
- **容器化**：Docker + Docker Compose
- **監控**：Prometheus + Grafana
- **日誌**：OpenTelemetry
- **反向代理**：Nginx

## 🔐 安全與合規

### 身份驗證
- **標準**：OIDC/OAuth2
- **JWT**：存取令牌管理
- **RBAC**：角色基礎存取控制

### 資料保護
- **加密**：AES-256 資料加密
- **假名化**：PHI 資料處理
- **審計**：完整操作日誌

### 合規標準
- **HIPAA**：醫療資料保護
- **TW PDPA**：台灣個資法
- **FHIR**：醫療資料交換標準

## 📊 監控與運維

### 系統監控
- **指標**：Prometheus 收集
- **視覺化**：Grafana 儀表板
- **告警**：閾值觸發通知

### 應用監控
- **效能**：API 回應時間
- **錯誤**：異常率追蹤
- **使用量**：LLM 呼叫統計

### 日誌管理
- **結構化**：JSON 格式日誌
- **集中化**：統一日誌收集
- **保留期**：90 天審計日誌

## 🚀 部署策略

### 開發環境
```bash
# 一鍵啟動
docker-compose -f infra/docker/docker-compose.dev.yml up -d
```

### 測試環境
- **自動化測試**：CI/CD 管道
- **整合測試**：API 端到端測試
- **效能測試**：負載測試

### 生產環境
- **容器編排**：Kubernetes
- **高可用**：多實例部署
- **備份**：定期資料備份

## 📈 開發流程

### 敏捷開發
- **Sprint**：2 週迭代
- **Backlog**：User Story 管理
- **驗收**：MVP 驗收流程

### 品質保證
- **程式碼審查**：Pull Request 審查
- **自動化測試**：單元 + 整合測試
- **安全掃描**：依賴項漏洞檢查

### 變更管理
- **CAB**：變更控制委員會
- **影響分析**：變更影響評估
- **回滾計畫**：緊急回滾機制

## 🎓 學習資源

### 文件
- [README.md](README.md) - 完整系統文件
- [QUICKSTART.md](QUICKSTART.md) - 快速開始指南
- [OpenAPI 規格](docs/openapi/openapi.yml) - API 文件

### Cursor Prompt
- [會議紀錄](docs/prompts/prompt_meeting_minutes.md) - 逐字稿轉換
- [KPI 監控](docs/prompts/prompt_kpi_daemon.md) - 指標追蹤
- [風險管理](docs/prompts/prompt_risk_matrix.md) - 風險評估
- [變更控制](docs/prompts/prompt_change_control.md) - 變更管理
- [MVP 驗收](docs/prompts/prompt_mvp_acceptance.md) - 驗收流程
- [Backlog 生成](docs/prompts/prompt_backlog.md) - 需求管理

### 技術資源
- [FastAPI 文件](https://fastapi.tiangolo.com/)
- [Next.js 文件](https://nextjs.org/docs)
- [Docker 文件](https://docs.docker.com/)
- [PostgreSQL 文件](https://www.postgresql.org/docs/)

## 🤝 貢獻指南

### 開發環境設置
1. 複製專案：`git clone <repository>`
2. 環境配置：`cp env.example .env`
3. 啟動服務：`docker-compose up -d`
4. 開始開發

### 提交規範
- **分支命名**：`feature/功能名稱` 或 `fix/問題描述`
- **提交訊息**：遵循 Conventional Commits
- **程式碼風格**：使用 Prettier + ESLint

### 測試要求
- **單元測試**：覆蓋率 > 80%
- **整合測試**：API 端到端測試
- **效能測試**：關鍵路徑負載測試

## 📞 聯繫資訊

- **專案負責人**：PM-AI Development Team
- **技術支援**：dev@pm-ai.com
- **文件問題**：docs@pm-ai.com
- **安全問題**：security@pm-ai.com

---

**最後更新**：2024年12月  
**版本**：1.0.0  
**狀態**：MVP 開發中
