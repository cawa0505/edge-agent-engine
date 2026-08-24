# EdgeAgent Engine 專案計畫書

> 專案代號：`edge-agent-d`  
> 定位：端側持續微調與推論引擎  
> 狀態：概念整理／技術可行性待驗證

## 1. 專案摘要

EdgeAgent Engine 是以 Rust 開發的端側 AI 系統服務，目標是在 Android、Linux 與其他 ARM64 邊緣裝置上，同時提供：

- 本機模型推論
- OpenAI-compatible API
- 訓練資料收集與排程
- LoRA／Adapter 增量微調
- Adapter 動態切換
- 電池、溫度及記憶體護欄

核心價值是讓敏感資料留在裝置內，並在符合資源條件時完成個人化訓練。引擎應與特定模型解耦，不綁定 Needle、SmolLM 或 Qwen 等單一模型。

## 2. 要解決的問題

目前端側 AI 工具多偏重靜態推論；常見微調框架則依賴 Python、伺服器級 GPU 與較大的執行環境。行動裝置上的持續訓練還需處理：

1. 推論與微調工具鏈分離。
2. 模型格式、架構與執行後端綁定。
3. Android 背景執行、電池與閒置限制。
4. 發熱、降頻及 Low Memory 終止風險。
5. 個人資料上傳雲端造成的隱私與合規疑慮。

## 3. 產品邊界

### 3.1 核心範圍

- Rust daemon 與可嵌入核心。
- HTTP、SSE 與 Unix Domain Socket。
- OpenAI-compatible 的基本推論介面。
- 本機資料佇列。
- 單一 Base Model 搭配多個 Adapter。
- 端側訓練排程與可中斷 checkpoint。
- Android／Linux 系統資源護欄。

### 3.2 非初期範圍

- 通用 Hugging Face 模型轉換平台。
- 支援所有 Transformer 架構。
- iOS `.xcframework`。
- 專屬 NPU／APU 最佳化。
- 視覺化 Dashboard 與雲端管理服務。
- 醫療、金融或 GDPR 合規認證。

以上功能待核心可行性成立後再評估，避免首版同時承擔模型框架、作業系統整合與商業平台三種風險。

## 4. 架構原則

### 4.1 模型與執行後端解耦

模型架構、權重格式、tokenizer、訓練能力及硬體 backend 不應混成單一介面。原始構想中的 `ModelArchitecture` Trait 可作為方向，但正式介面必須經第一個可運行模型驗證後再固定。

```rust
pub trait ModelArchitecture: Send + Sync {
    fn load_base(config: &ModelConfig, path: &Path) -> Result<Self>
    where
        Self: Sized;

    fn forward(&self, input_ids: &[u32], mask: Option<&Tensor>) -> Result<Tensor>;
    fn attach_adapter(&mut self, adapter: &AdapterWeights) -> Result<()>;
    fn detach_adapter(&mut self) -> Result<()>;
}
```

`backward` 與 optimizer 是否屬於同一 Trait 尚未決定。推論模型不一定具備 Auto-diff，強制放在同一介面可能造成錯誤抽象。

### 4.2 單一 Base Model、多 Adapter

- Base Model 權重盡量只保留一份。
- Adapter 以任務名稱尋址，例如 `edge:notification-classifier`。
- 切換成本與執行緒安全必須用 benchmark 驗證。
- 第一版不承諾「請求級毫秒熱切換」，只量測並公開實際結果。

### 4.3 系統護欄優先

訓練只能在策略允許時執行。建議初始條件：

- 已接上電源。
- 裝置處於閒置狀態。
- 電量高於設定門檻。
- 溫度低於裝置策略門檻。
- 可用記憶體高於安全水位。

所有門檻都需依裝置校正，不能把 `42°C` 或 `80%` 當成跨裝置安全常數。

## 5. 邏輯架構

```text
Applications / Local Agents
           │
           │ HTTP / SSE / Unix Domain Socket
           ▼
┌─────────────────────────────────────────────┐
│ EdgeAgent Daemon                            │
│                                             │
│  API Gateway                               │
│  ├─ /v1/chat/completions                   │
│  ├─ /v1/models                             │
│  └─ /v1/ingest                             │
│                                             │
│  Runtime                                   │
│  ├─ Model Registry                         │
│  ├─ Adapter Router                         │
│  ├─ Inference Session                      │
│  └─ Training Session                       │
│                                             │
│  System Services                           │
│  ├─ Scheduler                              │
│  ├─ Battery / Idle Guard                   │
│  ├─ Thermal / Memory Guard                 │
│  └─ Checkpoint Manager                     │
│                                             │
│  Persistence                               │
│  ├─ SQLite metadata / ingestion queue      │
│  └─ Model / Adapter / checkpoint files     │
└─────────────────────────────────────────────┘
```

## 6. 核心模組

### 6.1 API 與 IPC

- Axum + Tokio。
- `/v1/chat/completions`：基本非串流與 SSE 回應。
- `/v1/models`：列出可用 Base Model 與 Adapter。
- `/v1/ingest`：寫入本機待訓練資料。
- Unix Domain Socket：供同機程式使用。

「OpenAI-compatible」僅代表已實作端點與欄位的相容子集；發布時必須明列未支援功能。

### 6.2 Adapter Router

- 驗證 Adapter 與 Base Model 的相容性。
- 管理載入、卸載及生命週期。
- 防止並行請求互相污染 Adapter 狀態。
- 記錄版本、來源、訓練資料摘要與評估指標。

### 6.3 Ingestion 與本機儲存

- SQLite 管理資料佇列、Adapter metadata 與工作狀態。
- 模型及 checkpoint 使用檔案儲存，不塞入 SQLite BLOB。
- 敏感資料的加密、保留期限與刪除策略列為 trust boundary，不能省略。

### 6.4 持續微調

- 第一階段只支援一個已驗證的模型架構與一種 Adapter 方法。
- 訓練工作可暫停、恢復與取消。
- 每次升級 Adapter 前須保留上一個可用版本。
- 新 Adapter 必須通過最小評估，避免持續訓練造成品質退化。

### 6.5 系統護欄

- Android WorkManager／Foreground Service 的角色待實機驗證。
- Linux 可由 systemd timer、service 或 daemon 內部 scheduler 驅動。
- `/sys/class/thermal` 的可讀性與 sensor mapping 依裝置而異，需 fallback。
- Android Low Memory callback 只能提供預警，不能假設一定有足夠時間完成大型 checkpoint。

## 7. 技術選型

| 領域 | 候選技術 | 狀態 |
|---|---|---|
| 語言 | Rust 1.80+ | 確定方向 |
| Async runtime | Tokio | 待 PoC 驗證 |
| HTTP API | Axum | 待 PoC 驗證 |
| 張量／Auto-diff | Burn 或 Candle | `[待討論]`，不得同時導入 |
| Android binding | UniFFI 或 JNI | `[待討論]` |
| Android build | cargo-ndk | 待驗證 |
| Metadata／queue | SQLite + rusqlite | 建議 |
| 權重／Adapter | SafeTensors；GGUF 僅限可支援情境 | 待模型驗證 |
| 加速 | CPU／ARM NEON 優先 | 第一階段 |
| GPU | Vulkan／OpenCL | 非第一階段 |

## 8. 開發方法與里程碑

原始 12 週估算只能視為概念目標。端側 backward pass、Android 背景限制與裝置矩陣尚未驗證前，不宜承諾固定交付日。

### Phase 0：技術可行性驗證

- 選定一個可合法取得且可在 Rust 執行訓練的模型。
- 在 Linux ARM64 或 Android 裝置完成一次 forward、backward 與 Adapter 匯出。
- 測量 RAM 峰值、訓練時間、耗電及溫升。
- 確認 Burn 或 Candle 單一方案。

**Gate：** 實機能完成最小 Adapter 訓練，且資源數據可接受，才進入產品化。

### Phase 1：核心 Runtime

- 建立 Rust workspace。
- 實作單一模型 loader 與 tokenizer。
- 建立 inference／training session。
- 建立 Adapter 儲存格式及最小品質檢查。

**Gate：** 同一測試輸入可重現推論；訓練後 Adapter 可重新載入。

### Phase 2：API 與 Adapter Router

- 實作 `/v1/models`、`/v1/chat/completions` 與 SSE。
- 實作 Unix Domain Socket。
- 驗證 Adapter 切換正確性、延遲及並行隔離。

**Gate：** API contract test、並行測試與 benchmark 通過。

### Phase 3：Ingestion 與訓練工作流

- 建立 `/v1/ingest` 與 SQLite queue。
- 實作資料生命週期與刪除。
- 實作訓練 job 狀態、checkpoint、恢復及 rollback。

**Gate：** 強制中斷後可安全恢復；失敗 Adapter 不會取代現行版本。

### Phase 4：Android／Linux 系統整合

- 實作電池、充電、閒置、溫度及記憶體觀測。
- 建立節流與暫停策略。
- 完成 ARM64 交叉編譯及裝置測試。

**Gate：** 在指定測試裝置長時間運行，不造成不可接受的發熱、耗電或系統終止。

### Phase 5：SDK 與發布準備

- 封裝 Android AAR 或提供 daemon 安裝方式。
- 建立最小整合範例。
- 發布能力矩陣、benchmark、限制與安全說明。

**Gate：** 文件描述與實際能力一致，無誇大相容性或效能。

## 9. 驗證指標

每個支援裝置至少記錄：

- Base Model 與 Adapter 大小。
- 冷啟動與 warm inference latency。
- Adapter 切換延遲。
- 推論／訓練 RAM 峰值。
- 每個訓練 step 的時間。
- 固定時段的耗電與溫升。
- checkpoint 寫入時間與恢復成功率。
- 訓練前後任務品質。

不得以「毫秒級」、「不發燙」、「零成本」或「支援各種模型」作為未經量測的發布宣稱。

## 10. 商業模式假設

### 10.1 Open-core／雙重授權

Community 版本採 **Apache-2.0** 授權，企業取得商業授權。Apache-2.0 提供專利保護且相容 Rust／Android 生態，不因 copyleft 義務降低企業導入意願。

開源核心與閉源擴充的技術邊界需在 Phase 0 後具體劃定。

### 10.2 B2B SDK

可能客群：隱私筆記、企業通訊、醫療資料收集、金融工具、IoT 與專用 Android 裝置商。

收費單位可評估：

- 每 App／產品線。
- 每裝置啟用量。
- 年度企業授權與支援。
- 特定 SoC 最佳化專案費。

### 10.3 開發者工具

Desktop／Mobile Companion App 與模型轉換服務應在核心引擎有實際使用者後再建置，避免過早維護第二套產品。

## 11. 護城河假設

真正可能形成差異化的部分不是模型，而是：

- 跨裝置資源護欄與排程資料。
- 可中斷、可恢復、可 rollback 的端側訓練流程。
- Adapter 生命週期與品質防退化機制。
- Android／Linux 實機相容性矩陣。
- 特定 SoC backend 與長時間穩定性。

「第一個」、「完全合規」、「推論成本為零」及「Edge AI 時代的 SQLite」都屬宣傳假設，現階段不納入正式定位。

## 12. 主要風險

| 風險 | 影響 | 初步處理 |
|---|---|---|
| Rust backend 不支援目標模型訓練 | 核心不可行 | Phase 0 先做完整 backward PoC |
| 端側 RAM／耗電超標 | 無法產品化 | 先限制單一小模型並量測 |
| Android 背景工作被終止 | 訓練不可靠 | 短工作切片、checkpoint、可恢復 queue |
| Thermal sensor 不一致 | 護欄誤判 | 裝置策略與保守 fallback |
| Adapter 品質退化 | 使用者體驗變差 | 評估、版本化、rollback |
| Ingestion 涉及敏感資料 | 隱私與法規風險 | 加密、刪除、最小保留、權限隔離 |
| 第三方授權衝突 | 無法商用 | 發布前建立 dependency／model license 清單 |
| 過度承諾 model-agnostic | 維護成本失控 | 公開明確支援矩陣 |

## 13. 下一步

1. 完成模型、Rust backend 與 Android 訓練可行性研究。
2. 決定 Phase 0 唯一模型與唯一 tensor framework。
3. 定義實機與成功門檻。
4. 建立最小 PoC，不先做完整 API、Dashboard 或商業授權系統。
