### 目前暫定決策與理由

1. **第一個模型**: Needle 26M
   - 理由: 符合 Phase 0 低資源需求，需實測 RAM、Android 版本、儲存空間與熱/電量閾值

2. **測試裝置**: Poco X7 Pro / MediaTek Dimensity 8400
   - 留待實測：RAM 使用率、Android 系統版本、儲存空間、熱管理與電池效率

3. **授權**: Apache-2.0
   - 理由: 開源友好且符合社區授權需求

4. **第一個 Tensor Framework 候選**: Candle
   - 限制: 必須通過 Phase 0 推論 + 反向傳播 + LoRA 更新驗證，不能宣稱支援訓練

5. **Android 整合**: Kotlin Android Foreground Service + 最小 JNI 橋接
   - 使用: cargo-ndk 進行 ARM64 建置
   - UniFFI 留作 API 穩定後的 SDK 選項

6. **第一個執行環境**: 同一 Android 應用程序原生 Rust 程式庫
   - 不先實作 Python 執行環境、LibTorch 或分開 daemon/UDS IPC

7. **artefacts**: SafeTensors for 基底/適配器 tensors + 分開 JSON metadata
   - GGUF 只限未來推論/量化路徑，不作為訓練格式

8. **LibTorch/Python**: 開發/驗證備用方案

9. **未驗證假設與證據需求**:
   - [待驗證] Candle 訓練能力
   - [待討論] Android 綁定方式 (UniFFI/JNI)
   - [待驗證] 實機驗證數據 (RAM 尖峰、延遲、電量)

10. **研究備註**:
   - 過往代理報告含不可靠細節，禁止虛構 repo 參考或統計數據

--- 所有決策保持 [待驗證]/[待討論] 狀態，僅標示明確確定項目