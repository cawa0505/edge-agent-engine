## 1. Phase 0 Validation

- [ ] 1.1 Select one legally usable model and record architecture, tokenizer, weight format, and license; verify the decision is documented in `PROGRAM.md` and remains limited to one model
- [ ] 1.2 Evaluate Burn, Candle, and any LibTorch/FFI candidate against forward, backward, optimizer, adapter, checkpoint, and ARM64/Android requirements; verify the selected backend is supported by reproducible evidence before implementation
- [ ] 1.3 Validate the chosen backend and model with a Linux ARM64 or Android forward and backward run; verify the command, device, revision, and output are recorded
- [ ] 1.4 Measure peak RAM, training time, power use, and temperature during the Phase 0 run; verify raw measurements and device-specific thresholds are preserved

## 2. Runtime Core

- [ ] 2.1 Create the Rust workspace and separate model architecture, tokenizer, execution backend, adapter, training job, checkpoint, and system guard boundaries; verify the workspace builds and inference-only types do not require training methods
- [ ] 2.2 Implement one validated Base Model loader and inference session; verify repeated test input produces the documented reproducible output
- [ ] 2.3 Add backend capability negotiation for forward, backward, optimizer, checkpoint, and adapter export/reload; verify unsupported training requests fail before state mutation
- [ ] 2.4 Implement versioned adapter artifact metadata and SafeTensors interoperability checks; verify incompatible architecture, tensor name, dtype, or format versions are rejected without replacing the active adapter

## 3. API and Adapter Routing

- [ ] 3.1 Implement `/v1/models` and the documented OpenAI-compatible subset of `/v1/chat/completions`; verify unsupported fields return explicit errors
- [ ] 3.2 Implement `base_model:adapter_name` model resolution; verify unknown or incompatible selections leave the previous runtime state unchanged
- [ ] 3.3 Implement non-streaming and SSE completion responses plus Unix Domain Socket parity; verify contract tests observe equivalent response shapes
- [ ] 3.4 Implement adapter registration, compatibility checks, version retention, rollback, and concurrent request isolation; verify parallel requests never observe intermediate adapter state

## 4. Ingestion and Persistence

- [ ] 4.1 Implement `POST /v1/ingest` with dynamic `task_id`, `messages`, and `metadata` fields; verify valid payloads are queued without requiring a fixed prompt template
- [ ] 4.2 Reject malformed ingest payloads before persistence; verify missing `task_id`, invalid `messages`, and invalid message fields create no partial queue record
- [ ] 4.3 Implement durable SQLite queue metadata and separate model, adapter, and checkpoint file storage; verify FIFO records survive daemon restart and large binaries are not stored as SQLite blobs
- [ ] 4.4 Implement sensitive-data retention and deletion boundaries; verify deleted records and referenced files are no longer retrievable

## 5. Continuous Training

- [ ] 5.1 Implement schedulable, sliceable training jobs with pause, resume, and cancel states; verify guard violations prevent start or pause the job safely
- [ ] 5.2 Implement atomic per-step checkpoints and recovery; verify forced interruption resumes from the latest valid checkpoint without corrupting Base Model or existing adapters
- [ ] 5.3 Implement adapter evaluation, promotion, and rollback; verify a failed evaluation cannot replace the active adapter
- [ ] 5.4 Implement adapter export and reload; verify a valid artifact restores expected behavior and an incompatible artifact is rejected

## 6. System Guards and Status

- [ ] 6.1 Implement device-specific battery, charging, idle, thermal, and memory observation with unavailable-sensor fallbacks; verify policy thresholds are configurable per device and never universal constants
- [ ] 6.2 Implement training throttle, thermal pause, OOM protection, and conservative recovery; verify Base Model and existing adapters remain intact after each guard path
- [ ] 6.3 Implement `GET /v1/system/status` with `idle`, `training`, `paused_thermal`, and `paused_oom` states plus `thermal_level`, `active_adapter`, and `queue_pending_count`; verify each state reports a consistent snapshot

## 7. Integration Verification

- [ ] 7.1 Run API, persistence, adapter, checkpoint, and guard integration tests together; verify the documented failure paths do not lose data or corrupt active model state
- [ ] 7.2 Reproduce the complete Phase 0 acceptance flow on the target ARM64/Android path; verify forward, backward, adapter export/reload, interruption-safe checkpointing, and resource measurements are attached to the result
- [ ] 7.3 Review scope against `PROJECT_PLAN.md` and `PROGRAM.md`; verify no multi-model platform, full OpenAI API, Dashboard/SaaS, iOS, Vulkan/OpenCL/NPU, or unverified licensing claim was added
