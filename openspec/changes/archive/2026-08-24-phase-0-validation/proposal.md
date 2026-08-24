## Why

EdgeAgentDaemon 目前處於 Phase 0 文件化階段，尚未形成可執行的 OpenSpec 規格。本變更將為 Phase 0 驗證建立完整的行為規格，明確定義模型載入、推論 API、Adapter 管理、資料持久化、訓練流程及系統護欄的需求邊界。

## What Changes

- 建立 runtime-core 規格：定義模型架構與執行後端的解耦邊界
- 定義 inference-api 合約：HTTP/SSE 端點與 OpenAI-compatible 子集
- 規範 adapter-routing 行為：版本控制、相容性驗證、並行隔離
- 規定 local-data-persistence：SQLite 队列與敏感資料處理
- 規劃 continuous-training 工作流：中斷 checkpoint 與 rollback
- 制定 system-guards 介面：資源監控與保守回復

## Capabilities

### New Capabilities
- runtime-core: 模型載入、推論 session 管理
- inference-api: HTTP/SSE 端點定義
- adapter-routing: Adapter 版本控制與隔離
- local-data-persistence: 檔案儲存與敏感資料邊界
- continuous-training: 中斷 checkpoint 處理
- system-guards: 設備特定資源監控

## Impact

- 影響範圍：openspec/specs/、openspec/changes/phase-0-validation/
- 後續實作需遵循 Phase 0 驗證門檻
- 明確排除多模型、完整 API、Dashboard/SaaS、iOS 支援

## Open Questions

- [待討論] 第一個模型選擇 (Needle 26M vs SmolLM)
- [待討論] Tensor Framework (Candle vs Burn)
- [待討論] Android 綁定方式 (UniFFI/JNI)