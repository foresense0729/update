# RAGFlow 安裝與 BAAI-large-zh-v1.5 模型配置完整教學

本教學將引導您在 Windows 環境下使用 Docker Desktop 安裝 RAGFlow，並重點說明如何配置 BAAI-large-zh-v1.5 嵌入模型，以及解析您目前的自定義設定（`new-docker`）與官方預設值的差異。

## 1. 前置作業 (Prerequisites)

在開始之前，請確保您的系統已滿足以下需求：

1.  **作業系統**: Windows 10/11 (建議版本 21H2 或更高)。
2.  **核心元件**: 已啟用 WSL 2 (Windows Subsystem for Linux)。
    > [!NOTE]
    > 如果尚未啟用，請以管理員身分開啟 PowerShell 並執行 `wsl --install`，然後重新開機。
3.  **容器平台**: 已安裝 **Docker Desktop for Windows**。
    -   請至 [Docker 官網](https://www.docker.com/products/docker-desktop/) 下載並安裝。
    -   安裝後請啟動 Docker Desktop，並至 settings -> Resources -> WSL integration 確認已勾選您的 Linux 發行版（如 Ubuntu，如果有的話）。
4.  **記憶體**: 建議主機至少有 16GB RAM (因為 BAAI-large 模型與 RAGFlow 系統本身需要較多資源)。
    -   請在 `.wslconfig` 或 Docker Desktop 設定中分配足夠的記憶體給 WSL (建議至少 8GB-12GB)。

## 2. 下載與安裝 RAGFlow

### 步驟 2.1: 取得檔案
您已經完成了這一步：
1.  前往 [RAGFlow GitHub Release](https://github.com/infiniflow/ragflow/releases) 下載 `ragflow.zip` (對應 v0.22.x 版本)。
2.  解壓縮該檔案到您的工作目錄，例如 `C:\Users\howard\Desktop\ragflow-main`。

### 步驟 2.2: 準備配置檔案 (關鍵步驟)
> [!IMPORTANT]
> RAGFlow v0.22+ 為了縮減映像檔大小，預設**不再包含**嵌入模型 (Embedding Models)。您必須手動修改設定來下載並啟用它。

在您執行 `docker compose up` 之前，我們需要確保配置檔案正確。這也是您 `new-docker` 資料夾中修改的主要目的。

請確保您的 `docker` 目錄 (或您目前使用的目錄) 中的 `.env` 檔案包含以下設定：

1.  **啟用 Embedding 服務 Profile**:
    找到 `COMPOSE_PROFILES` 設定，確保 `tei-cpu` (如果您用 CPU 推論) 被啟用。
    ```bash
    # 在 .env 檔案中取消註解此行
    COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu
    ```

2.  **指定 BAAI-large-zh-v1.5 模型**:
    找到 `TEI_MODEL` 設定，將預設的 Qwen 模型改為您需要的模型。
    ```bash
    # 修改前
    # TEI_MODEL=${TEI_MODEL:-Qwen/Qwen3-Embedding-0.6B}
    
    # 修改後 (明確指定 BAAI large zh v1.5)
    TEI_MODEL=BAAI/bge-large-zh-v1.5
    ```
    *注意：RAGFlow 啟動時會自動從 HuggingFace 下載此模型，這需要一點時間，且需要網路連接。*

### 步驟 2.3: 啟動服務
1.  開啟 **PowerShell**。
2.  切換到您的 docker 目錄：
    ```powershell
    cd C:\Users\howard\Desktop\ragflow-main\ragflow-main\docker
    ```
    *(或者是您修改過的 `new-docker` 目錄)*
3.  啟動容器：
    ```powershell
    docker compose up -d
    ```
    - `-d` 表示在背景執行。
    - 第一次執行會下載 RAGFlow 映像檔和 Embedding 模型，請耐心等待。

### 步驟 2.4: 驗證模型下載
您可以使用以下指令查看 embedding 服務的日誌，確認模型是否下載完成：
```powershell
docker logs -f ragflow-docker-tei-1
```
*(容器名稱可能會根據您的資料夾名稱略有不同，可用 `docker ps` 查看)*
若看到 "Ready" 或類似訊息，表示模型已載入成功。

---

## 3. 解析自定義修改 (Why 'new-docker'?)

經過比對您提供的 `docker` (原始) 與 `new-docker` (您修改後) 資料夾，以下是您所做的修改及其原因的技術解析。這些修改是為了讓 RAGFlow 更適合您的本地開發環境。

### 3.1 解決端口衝突 (Port Conflicts)
**修改檔案**: `.env`
*   **原因**: Windows 本機通常已經佔用了 Port 80 (IIS 或其他 Web 服務) 和 Port 443。如果直接使用預設值，Docker 會因為端口被佔用而啟動失敗。
*   **您的修改**:
    ```ini
    # 原版
    SVR_WEB_HTTP_PORT=80
    SVR_WEB_HTTPS_PORT=443
    
    # 修改版 (將服務移至 8100/1443)
    SVR_WEB_HTTP_PORT=8100
    SVR_WEB_HTTPS_PORT=1443
    ```
*   **結果**: 您現在需要透過 `http://localhost:8100` 存取 RAGFlow。

### 3.2 強制使用中文優化模型
**修改檔案**: `.env`
*   **原因**: v0.22+ 預設模型 (Qwen) 雖然通用，但對於純中文環境，`BAAI/bge-large-zh-v1.5` 依然是目前的業界標準之一，效果通常更好。且預設配置沒有啟用任何 embedding profile。
*   **您的修改**:
    ```ini
    # 啟用 CPU 推論服務
    COMPOSE_PROFILES=${COMPOSE_PROFILES},tei-cpu
    
    # 指定特定模型
    TEI_MODEL=BAAI/bge-large-zh-v1.5
    ```

### 3.3 服務定義微調
**修改檔案**: `docker-compose.yml`
*   **原因**: 原版配置中包含了一些註解掉的 MCP Server 與 Admin Server 的啟動指令範例。
*   **您的修改**:
    -   移除了部分關於 `--enable-adminserver` 的繁雜註解，保持檔案乾淨。
    -   **重啟策略 (Restart Policy)**: 將 `restart: unless-stopped` 改為 `restart: on-failure`。這在開發環境比較合理，避免配置錯誤時 Docker 無限重試導致資源耗盡。

## 4. 這份配置的優點
這份修改後的配置 (`new-docker`) 是一份**高度客製化且正確**的 Windows 開發環境配置：
1.  **避開了常見坑**: 自動避開了 Windows Port 80 佔用的問題。
2.  **補全了缺失功能**: 手動補回了 v0.22 移除的 Embedding 功能，並選用了適合中文場景的模型。
3.  **資源節約**: 透過 `tei-cpu` 使用 CPU 進行 embedding，節省 GPU 資源（或適合無獨顯的筆電）。

## 5. 常見問題排除

-   **Q: 瀏覽器無法開啟 `http://localhost:8100`**
    -   檢查容器狀態: `docker ps -a`。確保 `ragflow-server` 狀態為 Up。
    -   檢查防火牆: 確保 Windows 防火牆沒有擋住 Docker 的網路橋接。

-   **Q: 上傳檔案一直顯示 Parsing**
    -   這通常是因為 Embedding 服務 (`tei`) 還沒準備好或記憶體不足崩潰。
    -   檢查日誌: `docker logs ragflow-docker-tei-1`。
    -   解決方法: 在 `.wslconfig` 增加記憶體限制，確保有足夠 RAM 載入 BAAI large 模型。

-   **Q: 資料夾內有看到 `.git` 或 `.gitignore` 檔案，我需要安裝 Git 嗎？**
    -   **不需要安裝額外套件**：這些只是 Git 版本控制系統的設定檔 (Metadata)，對於您執行 `docker compose up` 來說**完全沒有影響**。
    -   **補充說明**：
        -   `.gitignore`: 告訴 Git 哪些檔案不要上傳到雲端 (例如您下載的模型權重)。
        -   `.github`: 包含 GitHub 網站自動化流程的設定 (例如自動測試)。
        -   `.git` (隱藏資料夾): 紀錄程式碼變更歷史。
    -   只要您沒有要參與 RAGFlow 的程式碼開發或貢獻，**可以直接忽略它們**。

現在，您可以放心地使用您的 `new-docker` 配置來啟動 RAGFlow 了！
