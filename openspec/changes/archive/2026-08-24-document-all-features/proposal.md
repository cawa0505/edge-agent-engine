## Why

EdgeAgent Engine 目前只有產品計畫與 Phase 0 執行清單，尚未形成可供實作、驗證與後續維護使用的行為規格。這項變更將把端側推論、Adapter、持續微調、本機資料、系統護欄及 API／IPC 等核心能力寫成完整 OpenSpec 文件，讓後續技術選型與實作都有明確的需求邊界；尚未經實機驗證的模型、tensor backend、Android binding 與授權則維持待討論，不提前定案。

## What Changes

- 建立 EdgeAgent Runtime 的模型、推論 session、訓練 session 與能力邊界規格。
- 建立本機推論 API／IPC 規格，涵蓋 `/v1/models`、`/v1/chat/completions`、SSE 與 Unix Domain Socket 的相容子集。
- 建立 Base Model 與多 Adapter 的註冊、相容性驗證、載入、卸載、版本化及並行隔離規格。
- 建立 ingestion queue 與本機 metadata／檔案儲存規格，涵蓋敏感資料的保留、刪除與存取邊界。
- 建立持續微調工作流規格，涵蓋排程、暫停、恢復、取消、checkpoint、Adapter 匯出／重新載入、評估與 rollback。
- 建立 Android／Linux 系統護欄規格，涵蓋電源、閒置、電量、溫度、記憶體、節流與保守 fallback。
- 將 Phase 0 實機驗收條件與可重現量測要求寫入規格，包含 forward／backward、資源使用、耗電、溫升、checkpoint 與品質。
- 明確記錄初期不包含多模型平台、完整 OpenAI API、Dashboard／SaaS、iOS、Vulkan／OpenCL／NPU 與模型轉換服務。

## Capabilities

### New Capabilities

- `runtime-core`: 定義模型載入、推論 session、訓練 session，以及模型架構與執行 backend 的解耦邊界。
- `inference-api`: 定義 HTTP／SSE 與 Unix Domain Socket 的本機推論介面及 OpenAI-compatible 支援子集。
- `adapter-routing`: 定義 Base Model 搭配多 Adapter 的註冊、相容性、熱切換、版本與並行隔離行為。
- `local-data-persistence`: 定義 ingestion API、SQLite queue、metadata、檔案儲存、資料生命週期與敏感資料處理邊界。
- `continuous-training`: 定義端側 Adapter 微調工作、排程、可中斷 checkpoint、恢復、評估與 rollback 行為。
- `system-guards`: 定義 Android／Linux 電源、閒置、溫度、記憶體監控、節流、暫停與背景執行策略。

### Modified Capabilities

無。`openspec/specs/` 目前沒有既有 capability；本變更建立第一批規格。

## Impact

- 目前主要影響 `openspec/specs/`、`openspec/changes/document-all-features/` 及後續 Rust workspace 的模組邊界。
- 後續實作預計涉及 Rust runtime、模型／tensor backend、Axum／Tokio API、Unix Domain Socket、SQLite、Android／Linux 系統整合與 ARM64 交叉編譯。
- 技術選型仍以 Phase 0 驗證為準：Burn 或 Candle、UniFFI 或 JNI、第一個模型、第一台測試裝置及 Community license 均不得由本文件預先定案。
- 本變更本身不新增執行程式碼、不宣稱已支援任何模型或裝置，也不會建立 Dashboard、SaaS 或完整模型轉換平台。
