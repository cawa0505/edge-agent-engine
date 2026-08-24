# EdgeAgent Engine 執行計畫

## 專案資訊

- **名稱：** EdgeAgent Engine
- **執行檔：** `edge-agent-d`
- **狀態：** Discovery
- **目前階段：** Phase 0 技術可行性驗證
- **完整計畫：** [`PROJECT_PLAN.md`](PROJECT_PLAN.md)

## 目標

驗證小型語言模型能否在 Android／ARM64 裝置上，以 Rust 完成本機推論、Adapter 訓練、匯出與重新載入，並取得可用的 RAM、耗電、溫度與執行時間數據。

## 執行順序

### Discovery

- [ ] 研究 Burn 與 Candle 對目標模型、Auto-diff、LoRA、Android 的實際支援。
- [ ] 盤點候選模型的架構、權重格式、tokenizer 與授權。
- [ ] 研究 Android 背景執行、thermal API 與 Low Memory 限制。
- [ ] 建立目標裝置與量測方法。

### Plan

- [ ] 選定一個模型，不同時支援多模型。
- [ ] 選定 Burn 或 Candle，不建立雙 backend。
- [ ] 定義 Phase 0 的最小程式結構。
- [ ] 定義成功／停止條件與 benchmark 表格。

### Execute

- [ ] 在開發機完成 forward pass。
- [ ] 完成最小 Adapter backward pass。
- [ ] 匯出並重新載入 Adapter。
- [ ] 交叉編譯至 ARM64／Android。
- [ ] 在實機完成相同步驟。

### Verify

- [ ] 驗證訓練結果可重現。
- [ ] 記錄 RAM 峰值、執行時間、耗電及溫升。
- [ ] 驗證中斷不會損壞 Base Model 或既有 Adapter。
- [ ] 根據實測決定繼續、縮小範圍或停止。

## Phase 0 驗收條件

必須同時達成：

1. ARM64／Android 實機完成 forward 與 backward。
2. Adapter 可安全匯出、重新載入並影響預期輸出。
3. 有可重現的 benchmark 與測試命令。
4. 峰值記憶體未超過目標裝置可接受範圍。
5. 訓練工作可以切片或 checkpoint，不依賴單次長時間常駐。

## 待討論

- `[待討論]` 第一個模型：Needle 26M、SmolLM 135M 或其他已驗證候選。
- `[待討論]` Tensor framework：Burn 或 Candle。
- `[待討論]` 第一台 Android 測試裝置與可接受資源上限。
- 已定案 Community 授權採 Apache-2.0。
- `[待討論]` 開源核心與商業擴充的邊界。

## 暫不執行

- 完整 OpenAI API 相容層。
- 多模型通用 Trait 定案。
- Vulkan／OpenCL／NPU 最佳化。
- Dashboard、SaaS 與模型轉換平台。
- iOS SDK。

以上項目需等 Phase 0 通過後才排入。
