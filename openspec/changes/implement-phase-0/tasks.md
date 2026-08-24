## 1. Workspace Skeleton

- [x] 1.1 Create the Cargo workspace with `edge-agent-core`, `edge-agent-cli`, `edge-agent-jni` crates; verify `cargo build` passes on a fresh clone and no tensor logic exists outside `edge-agent-core`
- [ ] 1.2 Verify Needle 26M weight/tokenizer conversion to Candle-loadable SafeTensors; record the conversion command, artifact checksum, architecture, tokenizer, and license in `docs/PHASE0_EVIDENCE.md`; if conversion is blocked, record it and stop — this rejects the model candidate, not the task

## 2. Dev-Machine Forward

- [ ] 2.1 Implement SafeTensors base model load and CPU forward in `edge-agent-core`, exposed via `edge-agent-cli`; verify a fixed input twice produces identical tokens and record command + output
- [ ] 2.2 Build the Python/PyTorch reference on the dev machine for the same fixed input; verify the Candle output comparison records tolerance and divergence; failure blocks groups 3–5

## 3. Adapter Cycle

- [ ] 3.1 Implement LoRA adapter backward and update on the dev machine; verify Base Model weights are byte-identical before and after a training step
- [ ] 3.2 Implement adapter export (SafeTensors + sidecar JSON metadata) and reload; verify a reloaded adapter changes inference output and mismatched metadata is rejected without replacing the active adapter
- [ ] 3.3 Implement per-step atomic checkpoints with checksums; verify a kill -9 mid-training resumes from the latest valid checkpoint and a torn checkpoint is never selected

## 4. ARM64 / Android Build

- [ ] 4.1 Set up cargo-ndk and cross-compile `edge-agent-jni` to `arm64-v8a`; verify the produced `.so` loads in an Android app process without missing symbols
- [ ] 4.2 Implement the minimal JNI lifecycle (start, stop, status, checkpoint trigger); verify JNI contains no tensor or training logic and status reports a consistent snapshot

## 5. On-Device Validation

- [ ] 5.1 Run the full cycle (forward, reference tolerance where feasible, adapter cycle, checkpoint safety) on the Poco X7 Pro; verify each step meets the same acceptance as the dev machine
- [ ] 5.2 Record RAM peak, latency, training time, power, and temperature with raw samples and device-specific thresholds in `docs/PHASE0_EVIDENCE.md`
- [ ] 5.3 Write the stop/go decision with evidence links; on fail, execute the pre-agreed fallback (framework or model re-evaluation) instead of a workaround claim
