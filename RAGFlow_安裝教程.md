# RAGFlow 完整安裝與使用教程

> 本教程適用於 RAGFlow v0.22 及以後版本，特別說明如何配置 BAAI/bge-large-zh-v1.5 內嵌模型

## 目錄

1. [前置作業](#前置作業)
2. [下載與解壓縮 RAGFlow](#下載與解壓縮-ragflow)
3. [配置檔案修改詳解](#配置檔案修改詳解)
4. [內嵌模型配置](#內嵌模型配置)
5. [啟動 RAGFlow](#啟動-ragflow)
6. [首次登入與設定](#首次登入與設定)
7. [驗證與測試](#驗證與測試)

---

## 前置作業

### 系統需求

在開始安裝 RAGFlow 之前，請確認您的系統符合以下最低需求：

- **作業系統**: Windows 10/11、macOS、Linux
- **CPU**: 至少 4 核心 (x86 架構)
- **記憶體**: 至少 16GB RAM
- **硬碟空間**: 至少 50GB 可用空間
- **Docker**: Docker Desktop 24.0.0 或更高版本
- **Docker Compose**: v2.26.1 或更高版本

### Docker Desktop 安裝與設定

#### 1. 下載 Docker Desktop

前往 [Docker 官方網站](https://www.docker.com/products/docker-desktop/) 下載適合您作業系統的 Docker Desktop。

#### 2. 安裝 Docker Desktop

- **Windows**: 執行下載的 `.exe` 檔案，按照安裝精靈完成安裝
- **macOS**: 拖曳 Docker.app 到應用程式資料夾
- **Linux**: 按照官方文件進行安裝

#### 3. 啟動 Docker Desktop

安裝完成後，啟動 Docker Desktop 並等待其完全啟動（系統托盤圖示應顯示為綠色）。

#### 4. 驗證安裝

開啟 PowerShell (Windows) 或 Terminal (macOS/Linux)，執行以下命令驗證安裝：

```powershell
docker --version
docker compose version
```

應該看到類似以下輸出：

```
Docker version 24.0.0, build xxx
Docker Compose version v2.26.1
```

---

## 下載與解壓縮 RAGFlow

### 1. 從 GitHub 下載 RAGFlow

前往 [RAGFlow GitHub Repository](https://github.com/infiniflow/ragflow) 頁面：

1. 點擊綠色的 **Code** 按鈕
2. 選擇 **Download ZIP**
3. 將 `ragflow-main.zip` 下載到您的電腦

> **提示**: 建議將檔案下載到桌面或其他容易找到的位置

### 2. 解壓縮檔案

將下載的 `ragflow-main.zip` 解壓縮到您選擇的目錄。解壓縮後，您會看到以下主要目錄結構：

```
ragflow-main/
├── docker/              # Docker 相關配置檔案（重要！）
│   ├── .env
│   ├── docker-compose.yml
│   ├── docker-compose-base.yml
│   ├── nginx/
│   └── ...
├── web/                 # 前端網頁界面
├── api/                 # API 相關程式碼
├── rag/                 # RAG 核心邏輯
├── conf/                # 應用程式配置
└── ...
```

### 3. 進入 Docker 目錄

使用 PowerShell 進入解壓縮後的 `docker` 目錄：

```powershell
cd ~\Desktop\ragflow-main\ragflow-main\docker
```

> **重要**: 所有後續的 Docker 命令都必須在這個 `docker` 目錄下執行

---

## 配置檔案修改詳解

在啟動 RAGFlow 之前，需要修改三個關鍵配置檔案。以下將詳細說明每個檔案的修改內容及原因。

### 1. `.env` 檔案修改

`.env` 檔案包含 RAGFlow 的環境變數配置。以下是需要修改的關鍵項目：

#### 修改項目一覽表

| 配置項 | 原始值 | 修改後值 | 說明 |
|--------|--------|----------|------|
| `DOC_ENGINE` | `elasticsearch` | `infinity` | 文件引擎類型 |
| `DEVICE` | `cpu` | `gpu` | 運算設備類型 |
| `SVR_WEB_HTTP_PORT` | `80` | `8100` | Web HTTP 埠號 |
| `SVR_WEB_HTTPS_PORT` | `443` | `1443` | Web HTTPS 埠號 |
| `COMPOSE_PROFILES` | 註解 | 啟用 `tei-cpu` | Docker Compose Profile |
| `TEI_MODEL` | `Qwen/Qwen3-Embedding-0.6B` | `BAAI/bge-large-zh-v1.5` | 內嵌模型 |
| `USE_MINERU` | 註解 | `false` | MinerU 功能 |
| `MINERU_DELETE_OUTPUT` | 註解 | `0` | MinerU 輸出保留 |
| `MINERU_BACKEND` | 註解 | `pipeline` | MinerU 後端 |

#### 詳細修改說明

打開 `docker/.env` 檔案，進行以下修改：

##### 1.1 文件引擎設定

```env
# 原始值
DOC_ENGINE=${DOC_ENGINE:-elasticsearch}

# 修改為
DOC_ENGINE=${DOC_ENGINE:-infinity}
```

**修改原因**: Infinity 是更新且效能更好的文件引擎，提供更快的向量檢索速度。

##### 1.2 運算設備設定

```env
# 原始值
DEVICE=${DEVICE:-cpu}

# 修改為
DEVICE=${DEVICE:-gpu}
```

**修改原因**: 
- 如果您的電腦有 NVIDIA GPU 且已安裝 CUDA，使用 GPU 可大幅提升內嵌模型和推理速度
- 如果沒有 GPU，請保持原始值 `cpu`

##### 1.3 Web 服務埠號設定

```env
# 原始值
SVR_WEB_HTTP_PORT=80
SVR_WEB_HTTPS_PORT=443

# 修改為
SVR_WEB_HTTP_PORT=8100
SVR_WEB_HTTPS_PORT=1443
```

**修改原因**: 
- 預設的 80 和 443 埠可能已被其他服務佔用
- 使用非特權埠 (>1024) 可避免權限問題
- 8100 和 1443 是較不易衝突的埠號

##### 1.4 啟用 TEI CPU Profile

```env
# 原始值（被註解）
# COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu

# 修改為
COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu
```

**修改原因**: 
- 啟用 Text Embeddings Inference (TEI) CPU 服務
- v0.22 之後版本需要明確啟用此 profile 才能使用內嵌模型
- 如果使用 GPU，也可以啟用 `tei-gpu` 替代

##### 1.5 內嵌模型設定 ⭐ 重要

```env
# 原始值
TEI_MODEL=${TEI_MODEL:-Qwen/Qwen3-Embedding-0.6B}

# 修改為
#TEI_MODEL=BAAI/bge-large-zh-v1.5
TEI_MODEL=BAAI/bge-large-zh-v1.5
```

**修改原因**: 
- **v0.22 之後的重要變更**: RAGFlow 不再預設包含內嵌模型
- BAAI/bge-large-zh-v1.5 是針對中文優化的高品質內嵌模型
- 模型維度為 1024，適合中文文檔的語義理解
- 第一次啟動時會自動下載此模型（約 1.3GB）

> **注意**: 如果您主要處理英文文檔，可以使用 `BAAI/bge-large-en-v1.5` 或保持預設的 `Qwen/Qwen3-Embedding-0.6B`

##### 1.6 MinerU 配置（可選）

```env
# 啟用 MinerU（高品質 PDF 解析工具）
USE_MINERU=false
MINERU_EXECUTABLE="$HOME/uv_tools/.venv/bin/mineru"
MINERU_DELETE_OUTPUT=0   # 保留輸出目錄
MINERU_BACKEND=pipeline  # 使用 pipeline 後端
```

**修改原因**: 
- MinerU 提供更精確的 PDF 文檔解析
- `MINERU_DELETE_OUTPUT=0` 保留中間輸出，便於除錯
- 如果不需要可保持預設註解狀態

---

### 2. `docker-compose.yml` 檔案修改

`docker-compose.yml` 定義了 RAGFlow 的服務容器配置。

#### 修改項目一覽表

| 服務 | 配置項 | 原始值 | 修改後值 | 說明 |
|------|--------|--------|----------|------|
| `ragflow-server` | `restart` | `unless-stopped` | `on-failure` | 重啟策略 |
| `ragflow-server` | `command` | 註解 | 啟用 adminserver | 管理伺服器 |
| `ragflow-server` | `command` | 註解 | 啟用 mcpserver | MCP 伺服器 |

#### 詳細修改說明

打開 `docker/docker-compose.yml` 檔案：

##### 2.1 啟用 Admin Server

```yaml
# 在 ragflow-server 服務的 command 區段
# 原始值（被註解）
# Example configuration to start Admin server:
#   - --enable-adminserver

# 修改為
# Example configration to start Admin server:
    #   - --enable-adminserver
```

**修改原因**: 
- Admin Server 提供系統管理界面
- 可以監控系統狀態、管理用戶權限等
- 根據需求決定是否啟用

##### 2.2 啟用 MCP Server

```yaml
# 原始值（被註解）
#   - --enable-mcpserver
#   - --mcp-host=0.0.0.0
#   - --mcp-port=9382
#   - --mcp-base-url=http://127.0.0.1:9380
#   - --mcp-script-path=/ragflow/mcp/server/server.py
#   - --mcp-mode=self-host
#   - --mcp-host-api-key=ragflow-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 修改為（取消註解）
      - --enable-mcpserver
      - --mcp-host=0.0.0.0
      - --mcp-port=9382
      - --mcp-base-url=http://127.0.0.1:9380
      - --mcp-script-path=/ragflow/mcp/server/server.py
      - --mcp-mode=self-host
      - --mcp-host-api-key=ragflow-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**修改原因**: 
- MCP (Model Context Protocol) Server 提供外部整合介面
- 允許與其他應用程式進行通訊
- 適合需要 API 整合的場景

##### 2.3 修改重啟策略

```yaml
# 所有服務的 restart 設定
# 原始值
    restart: unless-stopped

# 修改為
    restart: on-failure
```

**修改原因**: 
- `on-failure`: 只在容器異常退出時重啟
- `unless-stopped`: 除非手動停止，否則一直重啟
- `on-failure` 更適合開發測試環境，避免問題容器持續重啟

---

### 3. `docker-compose-base.yml` 檔案修改

`docker-compose-base.yml` 定義了 RAGFlow 底層服務的詳細配置。

#### 修改項目一覽表

| 服務 | 配置項 | 原始值 | 修改後值 | 說明 |
|------|--------|--------|----------|------|
| 所有服務 | `restart` | `unless-stopped` | `on-failure` | 重啟策略 |
| `infinity` | `image` | `v0.6.11` | `v0.6.7` | Infinity 版本 |
| `tei-cpu/gpu` | `command` | `/data/${TEI_MODEL}` | `${TEI_MODEL}` | 模型路徑 |

#### 詳細修改說明

打開 `docker/docker-compose-base.yml` 檔案：

##### 3.1 修改所有服務的重啟策略

```yaml
# 範例：以 es 服務為例
es:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
  # 原始值
  restart: unless-stopped
  
  # 修改為
  restart: on-failure
```

**修改原因**: 與 `docker-compose.yml` 相同，統一使用 `on-failure` 策略。

##### 3.2 修改 Infinity 版本

```yaml
infinity:
  # 原始值
  image: infiniflow/infinity:v0.6.11
  
  # 修改為
  image: infiniflow/infinity:v0.6.7
```

**修改原因**: 
- v0.6.7 是經過充分測試的穩定版本
- v0.6.11 可能存在相容性問題
- 如遇到問題可嘗試回退版本

##### 3.3 修改 TEI 模型加載路徑 ⭐ 重要

```yaml
# tei-cpu 服務
tei-cpu:
  # 原始值
  command: ["--model-id", "/data/${TEI_MODEL}", "--auto-truncate"]
  
  # 修改為
  command: ["--model-id", "${TEI_MODEL}", "--auto-truncate"]

# tei-gpu 服務（如果使用 GPU）
tei-gpu:
  # 原始值
  command: ["--model-id", "/data/${TEI_MODEL}", "--auto-truncate"]
  
  # 修改為
  command: ["--model-id", "${TEI_MODEL}", "--auto-truncate"]
```

**修改原因**: 
- **v0.22 的關鍵變更**: TEI 容器會自動從 HuggingFace 下載模型
- 不再需要預先掛載本地模型目錄 `/data`
- 直接使用模型 ID（如 `BAAI/bge-large-zh-v1.5`）即可
- 首次啟動時會自動下載到容器內部

---

## 內嵌模型配置

### v0.22 之後的重要變更

從 RAGFlow v0.22 開始，Docker 映像檔採用 "slim" 版本，不再預先包含內嵌模型。這意味著：

1. **映像檔更小**: 基礎映像檔從數 GB 減少到約 1GB
2. **按需下載**: 首次啟動時才下載您指定的模型
3. **靈活配置**: 可以輕鬆切換不同的內嵌模型

### BAAI/bge-large-zh-v1.5 模型簡介

**BAAI/bge-large-zh-v1.5** 是由北京智源人工智能研究院 (BAAI) 開發的中文內嵌模型：

- **模型大小**: 約 1.3GB
- **向量維度**: 1024 維
- **語言支援**: 主要針對中文優化，也支持英文
- **適用場景**: 
  - 中文文檔檢索
  - 語義相似度計算
  - RAG (Retrieval-Augmented Generation) 應用

### 模型下載流程

當您首次啟動 RAGFlow 時，TEI 服務會自動執行以下步驟：

1. **檢查模型**: 檢查容器內是否已存在指定的模型
2. **連接 HuggingFace**: 連接到 HuggingFace Model Hub
3. **下載模型**: 下載 `BAAI/bge-large-zh-v1.5` 模型檔案（約 1.3GB）
4. **加載模型**: 將模型加載到記憶體中
5. **啟動服務**: TEI 服務準備就緒，可接受內嵌請求

> **注意**: 首次下載可能需要 10-30 分鐘，取決於您的網路速度。請耐心等待。

### 加速下載（可選）

如果您在中國大陸或網路連線不穩定，可以考慮：

#### 方法一: 使用 HuggingFace 鏡像站

修改 `docker-compose-base.yml` 中的環境變數：

```yaml
tei-cpu:
  environment:
    - HF_ENDPOINT=https://hf-mirror.com  # 使用鏡像站
```

#### 方法二: 預先下載模型到本地

如果您想完全離線部署：

1. 手動從 [HuggingFace](https://huggingface.co/BAAI/bge-large-zh-v1.5) 下載模型
2. 將模型放置在本地目錄（如 `./models/bge-large-zh-v1.5/`）
3. 在 `docker-compose-base.yml` 中掛載目錄：

```yaml
tei-cpu:
  volumes:
    - ./models:/data
  command: ["--model-id", "/data/bge-large-zh-v1.5", "--auto-truncate"]
```

---

## 啟動 RAGFlow

完成所有配置修改後，即可啟動 RAGFlow。

### 啟動命令

在 `docker` 目錄下，執行以下命令：

```powershell
docker compose up -d
```

**參數說明**:
- `docker compose`: Docker Compose v2 命令（舊版為 `docker-compose`）
- `up`: 啟動服務
- `-d`: 以背景模式運行（detached mode）

### 啟動過程

執行命令後，您會看到類似以下輸出：

```
[+] Running 10/10
 ✔ Network ragflow_ragflow        Created
 ✔ Container ragflow-mysql        Started
 ✔ Container ragflow-redis        Started
 ✔ Container ragflow-es           Started
 ✔ Container ragflow-infinity     Started
 ✔ Container ragflow-tei-cpu      Starting...
 ✔ Container ragflow-minio        Started
 ✔ Container ragflow-server       Started
```

### 檢查容器狀態

使用以下命令檢查所有容器是否正常運行：

```powershell
docker compose ps
```

所有容器的 STATUS 應該顯示為 `Up` 或 `Up (healthy)`，例如：

```
NAME                  STATUS
ragflow-mysql         Up (healthy)
ragflow-redis         Up (healthy)
ragflow-es            Up
ragflow-infinity      Up
ragflow-tei-cpu       Up
ragflow-minio         Up (healthy)
ragflow-server        Up
```

### 查看日誌

#### 查看所有服務日誌

```powershell
docker compose logs -f
```

#### 查看特定服務日誌

```powershell
# 查看 RAGFlow 主服務日誌
docker compose logs -f ragflow-server

# 查看 TEI 服務日誌（內嵌模型）
docker compose logs -f tei-cpu

# 查看 Infinity 日誌
docker compose logs -f infinity
```

### 首次啟動注意事項

**首次啟動時，請特別關注 `tei-cpu` 服務的日誌**：

```powershell
docker compose logs -f tei-cpu
```

您應該會看到模型下載的進度訊息：

```
Downloading model BAAI/bge-large-zh-v1.5...
Downloading (1/5): config.json
Downloading (2/5): model.safetensors
Downloading (3/5): tokenizer.json
...
Model loaded successfully
Server is ready to accept connections
```

> **注意**: 模型下載期間，tei-cpu 容器可能會顯示 `Restarting` 狀態，這是正常現象。等待下載完成後，容器會穩定在 `Up` 狀態。

---

## 首次登入與設定

### 訪問 RAGFlow Web 界面

模型下載完成且所有服務正常運行後，即可訪問 RAGFlow：

1. 打開瀏覽器
2. 前往: `http://localhost:8100`
3. 如果修改了 `SVR_WEB_HTTP_PORT`，請使用您設定的埠號

### 首次登入

RAGFlow 預設管理員帳號：

- **使用者名稱**: `admin`
- **密碼**: `admin`

> **安全提示**: 首次登入後，請立即修改預設密碼！

### 配置內嵌模型

登入後，需要在 RAGFlow 界面中配置內嵌模型：

#### 步驟 1: 進入模型提供商設定

1. 點擊右上角的個人頭像
2. 選擇 **Model Providers**（模型提供商）

#### 步驟 2: 添加 TEI 內嵌模型

1. 點擊 **Add LLM** 按鈕
2. 填寫以下資訊：
   - **Model Type**: 選擇 `Embedding`
   - **Model Name**: `bge-large-zh-v1.5`
   - **Base URL**: `http://tei-cpu:8080`（如果使用 GPU，則為 `http://tei-gpu:8080`）
   - **API Key**: 留空（本地部署不需要）
3. 點擊 **Save** 儲存

#### 步驟 3: 設定預設內嵌模型

1. 返回個人頭像選單
2. 選擇 **System Model Settings**（系統模型設定）
3. 在 **Embedding Model** 下拉選單中選擇 `bge-large-zh-v1.5`
4. 點擊 **Save** 儲存

### 配置 LLM（大型語言模型）

RAGFlow 還需要配置 LLM 來生成回答。您可以選擇：

#### 選項 1: 使用本地 LLM（Ollama）

1. 安裝 [Ollama](https://ollama.ai/)
2. 下載模型，例如：
   ```powershell
   ollama pull qwen2.5:7b
   ```
3. 在 RAGFlow 的 **Model Providers** 中添加 Ollama：
   - **Provider Type**: Ollama
   - **Base URL**: `http://host.docker.internal:11434`
   - **Model Name**: `qwen2.5:7b`

#### 選項 2: 使用雲端 API

也可以使用 OpenAI、Azure OpenAI、Google Gemini 等雲端服務的 API。

---

## 驗證與測試

### 驗證內嵌模型

#### 測試 TEI 服務

使用 curl 或 PowerShell 測試 TEI 服務是否正常：

```powershell
curl -X POST http://localhost:8080/embed `
  -H "Content-Type: application/json" `
  -d '{"inputs": "測試文本"}'
```

如果返回包含向量的 JSON 回應，表示內嵌模型工作正常。

### 建立測試知識庫

#### 步驟 1: 建立知識庫

1. 在 RAGFlow 首頁點擊 **Datasets**（資料集）
2. 點擊 **Create Dataset**（建立資料集）
3. 填寫資料集名稱，例如 "測試知識庫"
4. **重要**: 選擇 **Embedding Model** 為 `bge-large-zh-v1.5`
5. 選擇 **Chunking Method**（分塊方法），推薦使用 `General`
6. 點擊 **Create** 建立

> **注意**: 一旦選定內嵌模型並上傳文件，該資料集的內嵌模型就無法更改！

#### 步驟 2: 上傳測試文件

1. 進入剛建立的資料集
2. 點擊 **Upload** 按鈕
3. 選擇一個測試文件（支援 PDF、DOCX、TXT、Markdown 等格式）
4. 上傳後，點擊 **Parse**（解析）按鈕
5. 等待文件解析完成（可在 **Files** 頁面查看進度）

#### 步驟 3: 建立對話

1. 點擊左側選單的 **Chat**（對話）
2. 點擊 **Create Assistant**（建立助手）
3. 選擇剛才建立的資料集
4. 選擇 LLM 模型
5. 開始測試對話！

### 測試問答

在對話框中輸入與上傳文件相關的問題，例如：

```
問：[您上傳文件的相關問題]
```

如果系統能夠正確引用文件內容並生成回答，表示 RAGFlow 安裝成功且運作正常！

---

## 常見問題與疑難排解

### Q1: TEI 容器持續重啟

**原因**: 模型下載未完成或下載失敗

**解決方法**:
1. 查看 TEI 容器日誌：
   ```powershell
   docker compose logs -f tei-cpu
   ```
2. 如果是網路問題，可以使用 HuggingFace 鏡像站（參考前文）
3. 等待模型完全下載完成（可能需要 10-30 分鐘）

### Q2: 無法訪問 Web 界面

**可能原因**:
- 埠號衝突
- 容器未正常啟動

**解決方法**:
1. 檢查埠號是否被佔用：
   ```powershell
   netstat -ano | findstr :8100
   ```
2. 確認所有容器都在運行：
   ```powershell
   docker compose ps
   ```
3. 檢查 ragflow-server 日誌：
   ```powershell
   docker compose logs -f ragflow-server
   ```

### Q3: 記憶體不足

**原因**: 運行所有服務需要大量記憶體

**解決方法**:
1. 關閉不必要的應用程式
2. 增加 Docker Desktop 的記憶體限制：
   - 打開 Docker Desktop → Settings → Resources
   - 增加 Memory 至至少 8GB
3. 如果仍不足，考慮使用 CPU 版本替代 GPU 版本

### Q4: 模型下載速度慢

**解決方法**:
- 使用 HuggingFace 鏡像站（中國大陸用戶）
- 或預先下載模型到本地並掛載（參考前文）

### Q5: 文件解析失敗

**可能原因**:
- 文件格式不支援
- 文件損壞
- 解析服務異常

**解決方法**:
1. 確認文件格式在支援列表中
2. 嘗試重新上傳文件
3. 查看 ragflow-server 日誌排查錯誤

---

## 附錄：配置檔案修改總結

以下以左右對比的方式展示三個關鍵配置檔案的原始內容與修改後的內容。

---

### 一、`.env` 檔案修改對比

#### 修改項目 1: 文件引擎設定

| 原始值 ⬅️ | 修改後 ➡️ |
|---------|----------|
| `DOC_ENGINE=${DOC_ENGINE:-elasticsearch}` | `DOC_ENGINE=${DOC_ENGINE:-infinity}` |

#### 修改項目 2: 運算設備設定

| 原始值 ⬅️ | 修改後 ➡️ |
|---------|----------|
| `DEVICE=${DEVICE:-cpu}` | `DEVICE=${DEVICE:-gpu}` |

#### 修改項目 3: Web 服務埠號設定

| 配置項 | 原始值 ⬅️ | 修改後 ➡️ |
|--------|----------|----------|
| HTTP Port | `SVR_WEB_HTTP_PORT=80` | `SVR_WEB_HTTP_PORT=8100` |
| HTTPS Port | `SVR_WEB_HTTPS_PORT=443` | `SVR_WEB_HTTPS_PORT=1443` |

#### 修改項目 4: 啟用 TEI CPU Profile

| 原始值 ⬅️ | 修改後 ➡️ |
|---------|----------|
| `# COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu` | `COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu` |

#### 修改項目 5: 內嵌模型設定 ⭐ 重要

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```env
TEI_MODEL=${TEI_MODEL:-Qwen/Qwen3-Embedding-0.6B}
```

</td>
<td>

```env
#TEI_MODEL=BAAI/bge-large-zh-v1.5
TEI_MODEL=BAAI/bge-large-zh-v1.5
```

</td>
</tr>
</table>

#### 修改項目 6: MinerU 配置（可選）

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```env
# Uncommenting these lines will automatically 
# add MinerU to the model provider whenever possible.
# MINERU_DELETE_OUTPUT=0   # keep output directory
# MINERU_BACKEND=pipeline  # or another backend you prefer
```

</td>
<td>

```env
# Enable Mineru
USE_MINERU=false
MINERU_EXECUTABLE="$HOME/uv_tools/.venv/bin/mineru"
MINERU_DELETE_OUTPUT=0   # keep output directory
MINERU_BACKEND=pipeline  # or another backend you prefer
```

</td>
</tr>
</table>

---

### 二、`docker-compose.yml` 檔案修改對比

#### 修改項目 1: ragflow-server 重啟策略

| 原始值 ⬅️ | 修改後 ➡️ |
|---------|----------|
| `restart: unless-stopped` | `restart: on-failure` |

#### 修改項目 2: 啟用 Admin Server 與 MCP Server

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```yaml
ragflow-server:
  # ...其他配置
  restart: unless-stopped
  # Example configuration to start Admin server:
  command:
    - --enable-adminserver
  # MCP Server 配置被註解
  #   - --enable-mcpserver
  #   - --mcp-host=0.0.0.0
  #   - --mcp-port=9382
  #   - --mcp-base-url=http://127.0.0.1:9380
  #   - --mcp-script-path=/ragflow/mcp/server/server.py
  #   - --mcp-mode=self-host
  #   - --mcp-host-api-key=ragflow-xxxxxx...
```

</td>
<td>

```yaml
ragflow-server:
  # ...其他配置
  restart: on-failure
  # Example configration to start Admin server:
  #   - --enable-adminserver
  # MCP Server 配置啟用
  command:
    - --enable-mcpserver
    - --mcp-host=0.0.0.0
    - --mcp-port=9382
    - --mcp-base-url=http://127.0.0.1:9380
    - --mcp-script-path=/ragflow/mcp/server/server.py
    - --mcp-mode=self-host
    - --mcp-host-api-key=ragflow-xxxxxx...
```

</td>
</tr>
</table>

---

### 三、`docker-compose-base.yml` 檔案修改對比

#### 修改項目 1: 所有服務的重啟策略

| 原始值 ⬅️ | 修改後 ➡️ |
|---------|----------|
| `restart: unless-stopped` | `restart: on-failure` |

> 此修改適用於所有服務：`es`, `mysql`, `redis`, `minio`, `infinity`, `tei-cpu`, `tei-gpu` 等

#### 修改項目 2: Infinity 服務版本

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```yaml
infinity:
  image: infiniflow/infinity:v0.6.11
  restart: unless-stopped
  # ...其他配置
```

</td>
<td>

```yaml
infinity:
  image: infiniflow/infinity:v0.6.7
  restart: on-failure
  # ...其他配置
```

</td>
</tr>
</table>

#### 修改項目 3: TEI CPU 模型加載路徑 ⭐ 重要

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```yaml
tei-cpu:
  profiles: ["tei-cpu"]
  image: ghcr.io/huggingface/text-embeddings-inference:cpu-1.5
  restart: unless-stopped
  volumes:
    - ${HF_HOME:-$HOME/.cache/huggingface}:/data
  command: 
    - "--model-id"
    - "/data/${TEI_MODEL}"
    - "--auto-truncate"
  # ...其他配置
```

</td>
<td>

```yaml
tei-cpu:
  profiles: ["tei-cpu"]
  image: ghcr.io/huggingface/text-embeddings-inference:cpu-1.5
  restart: on-failure
  volumes:
    - ${HF_HOME:-$HOME/.cache/huggingface}:/data
  command: 
    - "--model-id"
    - "${TEI_MODEL}"
    - "--auto-truncate"
  # ...其他配置
```

</td>
</tr>
</table>

> **關鍵變更**: 模型路徑從 `/data/${TEI_MODEL}` 改為 `${TEI_MODEL}`，讓 TEI 自動從 HuggingFace 下載模型

#### 修改項目 4: TEI GPU 模型加載路徑 ⭐ 重要

<table>
<tr>
<th>原始值 ⬅️</th>
<th>修改後 ➡️</th>
</tr>
<tr>
<td>

```yaml
tei-gpu:
  profiles: ["tei-gpu"]
  image: ghcr.io/huggingface/text-embeddings-inference:1.5
  restart: unless-stopped
  volumes:
    - ${HF_HOME:-$HOME/.cache/huggingface}:/data
  command: 
    - "--model-id"
    - "/data/${TEI_MODEL}"
    - "--auto-truncate"
  # ...其他配置
```

</td>
<td>

```yaml
tei-gpu:
  profiles: ["tei-gpu"]
  image: ghcr.io/huggingface/text-embeddings-inference:1.5
  restart: on-failure
  volumes:
    - ${HF_HOME:-$HOME/.cache/huggingface}:/data
  command: 
    - "--model-id"
    - "${TEI_MODEL}"
    - "--auto-truncate"
  # ...其他配置
```

</td>
</tr>
</table>

> **關鍵變更**: 與 TEI CPU 相同，模型路徑改為直接使用環境變數，支持自動下載

---

## 結語

恭喜您完成 RAGFlow 的安裝與配置！現在您可以：

1. ✅ 上傳各種格式的文檔建立知識庫
2. ✅ 使用高品質的中文內嵌模型進行語義檢索
3. ✅ 透過對話界面與您的知識庫互動
4. ✅ 開發基於 RAG 的智慧應用

如果遇到任何問題，請參考：

- [RAGFlow 官方文檔](https://ragflow.io/docs)
- [GitHub Issues](https://github.com/infiniflow/ragflow/issues)
- [社群討論區](https://github.com/infiniflow/ragflow/discussions)

祝您使用愉快！🎉
