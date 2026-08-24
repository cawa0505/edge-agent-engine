## Context

The Phase 0 acceptance gate lives in main spec `phase-0-gate` (archived from the `phase-0-validation` change) but the repo has no executable code. This change creates the implementation sequence: a minimal Cargo workspace that validates Candle forward/backward on the dev machine first, then cross-compiles to Android ARM64 via cargo-ndk and repeats the cycle on the Poco X7 Pro. Prior research (Candle vs Burn, JNI binding) was directional only; no claim below is treated as verified until its task produces reproducible evidence.

## Goals / Non-Goals

**Goals:**
- Establish the minimal Rust workspace skeleton (`edge-agent-core`, `edge-agent-cli`, `edge-agent-jni`)
- Validate Candle dev-machine CPU forward for Needle 26M against a PyTorch reference
- Validate LoRA adapter backward/update/export/reload with interruption-safe checkpoints
- Produce an ARM64 `.so` loadable in an Android app process with a minimal JNI lifecycle
- Record reproducible RAM, latency, training-time, power, and temperature measurements on the target device

**Non-Goals:**
- No HTTP/OpenAI-compatible API, no SQLite ingestion queue, no multi-model abstraction (post-gate scope, owned by main specs `runtime-core` and siblings)
- No Burn fallback implementation; Burn is only re-evaluated if Candle fails the gate
- No Vulkan/OpenCL/NPU acceleration, no iOS, no Dashboard/SaaS
- No UniFFI; JNI-only in Phase 0
- No Python on device; Python is a dev-machine reference tool only

## Decisions

1. **Crate split: core / cli / jni**
   - *Rationale:* keeps tensor logic out of JNI; `edge-agent-jni` exposes only lifecycle and control APIs (start, stop, status, checkpoint trigger), matching architecture memory #5228
   - *Alternatives:* single crate (fast to start, but entangles JNI with tensor code)

2. **Dev-machine-first validation order**
   - *Rationale:* cheapest falsification first; if Candle cannot do forward+backward on x86_64 CPU, the ARM64 path is moot. Gate rule #5226 enforced: inference + backward + LoRA update must all pass before Candle is committed
   - *Alternatives:* straight to Android (slowest iteration loop)

3. **SafeTensors + sidecar JSON metadata for adapters**
   - *Rationale:* single artifact format decided in `docs/PHASE0_DECISIONS.md` (#5229); metadata (base_model_id, adapter_id, architecture, tensor names, dtype, rank, target modules, tokenizer revision, format version, checksum) lives in JSON beside the tensors
   - *Alternatives:* GGUF (inference-only path, #5230)

4. **Checkpoints are per-step atomic writes with checksum**
   - *Rationale:* interruption safety is a Phase 0 gate requirement; a torn checkpoint must never replace the last valid one

## Risks / Trade-offs

- [Risk: Candle training capability unproven] → Mitigation: gate order puts backward/LoRA update before any Android work; failure triggers framework re-evaluation, not workaround
- [Risk: Needle 26M lacks ready Candle implementation] → Mitigation: task 1.2 verifies model format conversion first; a conversion gap is recorded and may reject the model candidate
- [Risk: cargo-ndk/NDK toolchain friction] → Mitigation: Android build tasks isolated last; dev-machine gate results stand on their own
- [Risk: measurement noise on device] → Mitigation: record raw samples plus device-specific thresholds; no universal constants

## Migration Plan

1. Workspace skeleton + model record check (task group 1)
2. SafeTensors load + dev CPU forward + reference comparison (group 2)
3. Adapter backward/update/export/reload + checkpoint safety (group 3)
4. ARM64 cross-build + JNI lifecycle (group 4)
5. On-device full cycle + measurements + gate review (group 5)

## Open Questions

- `[待驗證]` Candle backward/optimizer maturity for the Needle architecture
- `[待驗證]` Needle 26M weight/tokenizer conversion path to Candle
- `[待驗證]` Poco X7 Pro thermal/battery sampling surface
