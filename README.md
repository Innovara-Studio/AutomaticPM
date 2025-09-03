# 糖腎共病衛教照護系統（PM-AI）— System Blueprint

## 0. 目標與範圍（Scope）

**目標一**：建置 QA 衛教機器人，支援護理人員遠距衛教、降低時間成本。

**目標二**：將護理對話轉為結構化護理紀錄與自動報告（可匯出/串接 EMR/FHIR）。

**交付策略**：以 MVP 驗證核心價值、取得用戶回饋，再迭代擴充（含驗收、簽署與檢驗流程）。

## 1. 角色與敏捷作業（Agile Ops）

**角色**：PO / SM / Dev Team（前後端、NLP、QA、設計）與臨床利害關係人。

**啟動與 Backlog 建立**：Persona、User Journey、User Story、AC、KPI、優先級；支援 Notion/Jira。

**技術評估與資源初估**：Tech Stack、技能盤點、Burn-down、KPI 合理性檢核清單。

**會議自動化**：逐字稿 ⇒ 會議紀錄兩階段、名詞標準化、台灣醫療用語規範。

**KPI 日更與風險通報**：JSON 輸入、偏移計算、超閾值即觸發風險通知。

**MVP 驗收**：Demo、測試、問題台帳、驗收報告與簽署、交付確認單。

**風險與變更管控**：風險矩陣、RACI、CAB、變更影響分析表與 API Diff。

## 2. 系統整體架構（Architecture）

### 2.1 分層

- **Client（Web）**：Next.js/React + Tailwind，含護理師工作台、病人衛教介面、報告檢視
- **API（Service）**：FastAPI（Python），REST/OpenAPI，集中權限、審計、日誌、RBAC
- **LLM/RAG**：
  - 在地模型：MedGemma / GPT-OSS 20B 推論服務（支援 OpenAI-compatible 端點）
  - RAG：衛教知識庫（糖腎共病）、指南與FAQ向量索引，含檢索重排序、答案引用來源
- **FHIR/EMR Adapter**：FHIR TW Core 對接、就診/檢驗/用藥摘要讀取、對話紀錄存檔（不存 PHI 原文）
- **Storage**：PostgreSQL（事務資料）、S3/MinIO（檔案）、向量庫（pgvector / Qdrant）
- **Observability**：OpenTelemetry、Prometheus、Grafana；LLM 回應品質與延遲監測
- **Security**：OIDC/OAuth2、作用域權限、加密（At-rest/ In-transit）、存取稽核與留痕

### 2.2 主要模組

- **Edu-QA Bot**：檢索（RAG）→ LLM 生成 → 引用來源 → 風險字詞過濾 → 回應（文字/語音）
- **Nurse Dialog Capture**：Websocket 串流、ASR（可後續接）、多輪語境保存、事件化（event sourcing）
- **Report Generator**：將情境 → 結構化欄位 → 報表（PDF/HTML/CSV）→ 匯出/簽核/版本控管
- **KPI & Risk Daemon**：每日讀 stand-up JSON 與 Dashboard 基準值 → 自動產摘要/風險
- **Change/Risk Console**：R-1∼R-5、C-1∼C-5 Prompt 範本接入，半自動產出審查文件

## 3. 資料流（Dataflow）

```mermaid
flowchart LR
  U[使用者(護理師/病人)]-->W[Web 前端]
  W--REST/WebSocket-->A[FastAPI 服務層]
  A--檢索-->V[向量庫/知識庫]
  A--推論-->L[LLM 服務(在地/相容API)]
  A--寫入-->DB[(PostgreSQL)]
  A--檔案-->S[(S3/MinIO)]
  A--FHIR-->F[FHIR/EMR Adapter]
  A--事件-->K[KPI/Risk Daemon]
  K--報表/通知-->W
```

## 4. 資料模型（重點表）

```sql
-- 衛教知識文件
CREATE TABLE edu_doc (
  doc_id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  tags TEXT[],
  source_url TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 對話事件
CREATE TABLE convo_event (
  event_id BIGSERIAL PRIMARY KEY,
  session_id UUID NOT NULL,
  role TEXT CHECK (role IN ('nurse','patient','bot')),
  content TEXT NOT NULL,
  ts TIMESTAMPTZ DEFAULT now()
);

-- 報告主檔
CREATE TABLE nurse_report (
  report_id UUID PRIMARY KEY,
  session_id UUID NOT NULL,
  patient_ref TEXT,     -- 可用 pseudonym/哈希，不落 PHI
  summary TEXT,
  structured_json JSONB, -- 指標/量表/建議
  status TEXT CHECK (status IN ('draft','signed','archived')),
  created_by TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## 5. API 合約（OpenAPI 節錄）

```yaml
paths:
  /api/qa/ask:
    post:
      summary: 提問衛教問題
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                question: { type: string }
                session_id: { type: string, format: uuid }
      responses:
        "200":
          description: 回答與引用
          content:
            application/json:
              schema:
                type: object
                properties:
                  answer: { type: string }
                  citations:
                    type: array
                    items: { type: object, properties: { doc_id: {type: string}, title: {type: string}, url: {type: string} } }

  /api/report/generate:
    post:
      summary: 由對話生成護理報告
      responses:
        "200": { description: JSON 結構 + 可下載連結 }
```

## 6. LLM/RAG 策略

**檢索**：多路徑檢索（BM25 + 向量）、醫療術語擴充（SNOMED CT / ICD-10 / LONIC 對應字典可後續加入）。

**生成**：系統提示包含「台灣醫療用語規範、不得出現對岸用語、不得保留口語贅詞」；逐字稿→會議紀錄兩段式。

**品質控管**：回答必附引用；對高風險語句標記並要求人工覆核；Daily KPI 監控模型延遲與答覆命中率。

**MVP 核心**：先做「糖腎共病」範圍的 FAQ 與報告生成，依驗收清單簽署。

## 7. 安全與合規（PHI/隱私/稽核）

**存取**：OIDC/OAuth2、RBAC、審計日誌。

**資料**：PHI 假名化、最小化原文存留、備援與復原演練。

**流程**：會議錄音/逐字稿自動標註敏感資訊，匯出前再次檢核（與「逐字稿規範」一致）。

**驗收**：依 MVP 驗收單與問題台帳過版；變更走 CAB 與影響分析。

## 8. Dev/MLOps

### 8.1 專案骨架

```
pm-ai/
 ├─ apps/
 │   ├─ web/                # Next.js (護理師/病人介面)
 │   └─ api/                # FastAPI
 ├─ services/
 │   ├─ llm_gateway/        # OpenAI-compatible 代理, 本地/遠端切換
 │   ├─ rag_indexer/        # 文件清洗/嵌入/索引
 │   └─ kpi_daemon/         # KPI/Risk 自動摘要
 ├─ adapters/
 │   └─ fhir/               # FHIR Client & Mapper
 ├─ data/
 │   ├─ seeds/              # 衛教內容/FAQ 初始資料
 │   └─ migrations/
 ├─ infra/
 │   ├─ docker/             # docker-compose.*.yml
 │   └─ k8s/                # 部署樣板
 ├─ docs/
 │   ├─ prd/                # PRD, Roadmap, MVP, 驗收單
 │   ├─ prompts/            # R-* / C-* / Meeting-KPI 模版
 │   └─ openapi/
 └─ .env.example
```

### 8.2 開發環境啟動

```bash
# 複製環境變數
cp .env.example .env

# 啟動開發環境
docker-compose -f infra/docker/docker-compose.dev.yml up -d

# 初始化資料庫
cd apps/api && python -m alembic upgrade head

# 啟動前端開發伺服器
cd apps/web && npm run dev
```

### 8.3 CI 檢查

Lint/Type Check、單元/整合測試、OpenAPI 驗證、容器鏡像掃描、基準負載測試。

## 9. 敏捷產物樣板

### 9.1 初版 Product Backlog（節錄）

| # | User Story | Acceptance Criteria | Priority | KPI 建議 |
|---|---|---|---|---|
| 1 | 作為護理師，我要能向 QA 機器人提問糖腎衛教問題，以便快速取得一致答案 | (1) 命中常見題 80% 以上；(2) 回覆＜3 秒 p90；(3) 答覆附引用 | High | 回答正確率、p90 延遲 |
| 2 | 作為護理師，我要能把對話生成護理報告，以便縮短紀錄時間 | (1) 生成 JSON+PDF；(2) 欄位完整率≧95%；(3) 可再編修與簽核 | High | 報告產出時間、欄位完整率 |
| 3 | 作為管理者，我要每日看到 KPI 摘要與風險通知 | (1) JSON 輸入計算；(2) 超閾值自動通知；(3) 可追溯 | Medium | KPI 偏移日數、通知反應時間 |

### 9.2 Cursor 內建 Prompt 使用

專案包含 6 個內建 Prompt，位於 `docs/prompts/`：

1. **逐字稿→會議紀錄**：兩階段轉換，符合台灣醫療用語規範
2. **KPI 摘要與風險通知**：自動分析 JSON 輸入，產出摘要與風險警告
3. **技術風險矩陣**：R-1～R-5 風險評估範本
4. **變更控制**：C-1～C-5 變更評估流程
5. **MVP 驗收**：功能測試、Demo、問題台帳、簽署流程
6. **Backlog 生成**：Persona → User Story → AC → KPI 自動化

**使用方式**：
- 在 Cursor 中直接呼叫對應的 Prompt 檔案
- 或將內容複製到 Cursor 的內建 Prompt 設定中
- 支援批量處理和自動化工作流程

## 10. 前後端實作要點

### 10.1 Web（Next.js）

**頁面**：登入、護理師工作台（對話/檢索/報告）、報告列表/檢視、管理儀表板。

**元件**：ChatPanel、CitationList、ReportEditor、KPIWidget、ChangeForm。

### 10.2 API（FastAPI）

**路由群**：/qa/*、/convo/*、/report/*、/fhir/*、/kpi/*、/change/*。

**中介層**：RBAC、審計 logging、速率限制、錯誤分類。

### 10.3 RAG Pipeline

**ingest**：清洗 → 片段化（醫療斷句）→ 嵌入 → upsert。

**answer**：檢索 → 重排序 → 提示組裝（內含台灣用語規則/不留贅詞）→ 生成 → 引用與過濾。

## 11. 測試策略

單元（路由/服務/提示模板）、整合（RAG 命中率、回覆延遲）、UAT（護理師腳本）、安全測試（XSS/SQLi/SSRF）。

基準：AC 對照表、MVP 驗收清單與簽署。

## 12. 上線與運維

**環境**：dev / staging / prod；金絲雀發布。

**監控**：API p95、LLM 延遲、RAG 命中率、錯誤率、成本（推論資源用量）。

**變更**：走 C-1∼C-5 的變更台帳與 CAB；若影響 API，生成 Diff 表。

## 13. 實作順序（MVP 三週版本）

**週1**：骨架（API/Web/DB/向量庫），ingest 最小版，/qa/ask 回傳固定答案（打通端到端）。

**週2**：接入在地 LLM + RAG，完成報告生成 JSON/PDF；導入逐字稿→紀錄流程。

**週3**：KPI Daemon + Dashboard；UAT + 驗收簽署；建立變更與風險作業。

## 14. 快速開始

1. **環境準備**：
   ```bash
   git clone <repository>
   cd pm-ai
   cp .env.example .env
   ```

2. **啟動開發環境**：
   ```bash
   docker-compose -f infra/docker/docker-compose.dev.yml up -d
   ```

3. **初始化資料**：
   ```bash
   cd apps/api && python -m alembic upgrade head
   cd apps/web && npm install && npm run dev
   ```

4. **使用 Cursor Prompt**：
   - 開啟 `docs/prompts/` 中的任一 Prompt 檔案
   - 複製內容到 Cursor 的內建 Prompt 設定
   - 開始使用自動化工作流程

## 15. 貢獻指南

1. 遵循敏捷開發流程
2. 使用內建 Prompt 進行文件產出
3. 所有變更需通過 CAB 審查
4. 保持 KPI 追蹤與風險監控

---

**專案狀態**：MVP 開發中  
**最後更新**：2024年12月  
**維護團隊**：PM-AI Development Team
