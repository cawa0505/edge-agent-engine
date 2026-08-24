# phase-0-implementation

## Purpose

Define the executable implementation sequence for the Phase 0 gate: a minimal Rust workspace that validates forward, adapter training cycle, checkpoint safety, and on-device measurement for one model, one backend, and one Android device. Every task must leave reproducible evidence; no capability is claimed before its evidence exists.

## ADDED Requirements

### Requirement: Minimal Rust workspace

The Phase 0 implementation MUST provide a Cargo workspace with separated crate boundaries before any tensor work.

#### Scenario: Workspace builds

When `cargo build` runs on a fresh clone, the workspace MUST compile with three crates: `edge-agent-core`, `edge-agent-cli`, `edge-agent-jni`. THEN no crate except `edge-agent-core` may contain tensor computation or training logic.

#### Scenario: Model and backend record

When the first forward runs, the model record (architecture, tokenizer, weight format, license) and backend revision MUST be written to `docs/PHASE0_EVIDENCE.md`. THEN the record identifies exactly one model and one backend.

### Requirement: Dev-machine CPU forward

The implementation MUST load the Phase 0 model from SafeTensors and run a CPU forward pass on the dev machine with deterministic output.

#### Scenario: Deterministic forward

When a fixed input is fed to the dev-machine forward twice, the token output MUST be identical. THEN the output and command are recorded in `docs/PHASE0_EVIDENCE.md`.

#### Scenario: Reference comparison

When a Python/PyTorch reference for the same input is available, the Candle output MUST be compared against it. THEN the comparison records tolerance and divergence; failure blocks all later groups.

### Requirement: Adapter training cycle

The implementation MUST perform a LoRA adapter backward, update, export, and reload cycle on the dev machine.

#### Scenario: Backward updates adapter only

When a training step executes, updated parameters MUST be adapter parameters only. THEN the Base Model weights are byte-identical before and after.

#### Scenario: Export and reload round-trip

When an adapter is exported to SafeTensors with sidecar JSON metadata and reloaded, inference output MUST reflect the adapter's effect. THEN an adapter with mismatched metadata (architecture, tensor names, dtype, or format version) is rejected without replacing the active adapter.

### Requirement: Interruption-safe checkpoint

The training loop MUST survive interruption without corrupting the Base Model or the last valid checkpoint.

#### Scenario: Kill and resume

When the training process is killed mid-step and restarted, it MUST resume from the latest valid checkpoint. THEN a torn checkpoint is never selected, verified by checksum.

### Requirement: ARM64 build and JNI lifecycle

The implementation MUST cross-compile via cargo-ndk to Android ARM64 and expose a minimal JNI bridge.

#### Scenario: cargo-ndk build

When `cargo ndk --target arm64-v8a` builds the library, it MUST produce a loadable `.so`. THEN the `.so` loads in an Android app process without missing-symbol errors.

#### Scenario: JNI lifecycle

When the Android app calls start/stop/status through JNI, the runtime MUST initialize, release resources, and report a consistent status snapshot. THEN the JNI surface contains no tensor or training logic.

### Requirement: On-device Phase 0 cycle

The full validation cycle MUST be reproduced on the Poco X7 Pro with recorded measurements.

#### Scenario: Device cycle

When the forward, adapter cycle, and checkpoint safety run on-device, each MUST pass the same acceptance as the dev machine. THEN RAM peak, latency, training time, power, and temperature are recorded with raw samples and device-specific thresholds in `docs/PHASE0_EVIDENCE.md`.

### Requirement: Gate review

The Phase 0 result MUST end in an explicit stop/go decision.

#### Scenario: Gate pass

When all prior requirements pass with evidence, the change is marked passed. THEN productization changes may be proposed.

#### Scenario: Gate fail

When any requirement fails, the failure MUST be recorded with root cause. THEN the pre-agreed fallback (framework or model re-evaluation) runs; no workaround claim is made.

## Scenarios

#### Scenario: No unverified claims

When evidence for a requirement does not exist, the capability MUST remain marked `[待驗證]` in docs. THEN no README, PROGRAM.md, or spec text claims it as supported.

#### Scenario: Single path enforcement

When any group is in progress, scope MUST remain one model, one backend, one device. THEN multi-model or multi-backend additions are rejected in review.

## Related

- `phase-0-gate` (main spec): the acceptance criteria this implementation satisfies.
- `runtime-core`, `continuous-training`, `system-guards` (main specs): post-gate productization capabilities explicitly out of scope here.
