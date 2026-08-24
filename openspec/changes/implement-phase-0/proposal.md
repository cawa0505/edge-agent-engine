## Why

Phase 0 驗收門檻已定義於主規格 `phase-0-gate`（原 `phase-0-validation` 變更已封存），但 repo 仍沒有任何可執行程式。本變更建立 Phase 0 的實作工作順序：先在開發機以最小 Rust workspace 驗證 Candle 的 forward 與 backward，再交叉編譯至 ARM64/Android 實機量測。先前調研僅為方向性結論，未經實證，因此實作順序必須把最便宜的驗證（開發機 CPU forward）排在最前面。

## What Changes

- 新增 Cargo workspace 與三個 crate：`edge-agent-core`、`edge-agent-cli`、`edge-agent-jni`
- 實作 Needle 26M 的 SafeTensors 載入與開發機 CPU forward（Candle）
- 建立 Python/PyTorch reference comparison 與容差記錄
- 實作 LoRA adapter backward/update/export/reload 循環
- 實作中斷安全 checkpoint
- 以 cargo-ndk 建置 ARM64 `.so` 並暴露最小 JNI lifecycle
- 在 Poco X7 Pro 實機完成同一循環並記錄 RAM、延遲、訓練時間、電量、溫度
- 所有量測與證據落地 `docs/`，可重現

## Capabilities

### New Capabilities
- phase-0-implementation: Phase 0 可執行驗證順序（workspace、forward、reference、adapter cycle、checkpoint、JNI、實機量測、證據記錄）

## Impact

- 首次引入程式碼：`Cargo.toml`、`crates/`、`rust-toolchain.toml`
- 依賴僅限：candle、safetensors、serde、tokenizers、jni（Android crate）
- 不含 HTTP API、SQLite 佇列、多模型抽象、Dashboard——這些屬 gate 後的產品化範圍（主規格 `runtime-core` 等 capability）

## Open Questions

- 無新增；Candle 訓練能力仍為 `[待驗證]`，由本變更的 gate 順序決定去留
