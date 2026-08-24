# phase-0-gate

## Purpose

Define the executable Phase 0 acceptance gate for EdgeAgentDaemon: a single-model, single-backend, single-Android-device validation path that must pass before any productization work begins.

## ADDED Requirements

### Requirement: Phase 0 model record

The Phase 0 model MUST be a single, legally usable model with recorded architecture, tokenizer, weight format, and license.

#### Scenario: Model metadata record
When the Phase 0 model is selected, its architecture, tokenizer type, weight format, and license MUST be recorded in `PROGRAM.md`. THEN the same record must be reproducible by a future reviewer without external lookup.

- The model MUST be limited to one candidate (Needle 26M) until Phase 0 passes.
- The license MUST be Apache-2.0 or otherwise verified as legally usable for local inference and adapter training.
- The weight format MUST be SafeTensors for base and adapter tensors, with separate JSON metadata.
- GGUF is NOT assumed to support training; it remains a future inference/quantization path.

### Requirement: Candle CPU forward

The selected backend MUST perform a CPU forward pass on the dev machine and on Android ARM64.

#### Scenario: Dev machine forward
When Candle runs a forward pass on the dev machine, it MUST produce deterministic output for a fixed input. THEN the output must match a recorded reference within numerical tolerance.

#### Scenario: Android ARM64 forward
When Candle runs a forward pass on Android ARM64, it MUST produce output equivalent to the dev machine result. THEN the same input must yield the same token sequence within numerical tolerance.

### Requirement: Reference comparison

The Phase 0 run MUST be compared against Python/PyTorch where available.

#### Scenario: PyTorch reference
When a PyTorch reference implementation is available, the Candle forward output MUST be compared against it. THEN the comparison must be recorded with tolerance and any divergence documented.

### Requirement: Adapter backward and update

The runtime MUST support a minimal adapter backward/update/export/reload cycle.

#### Scenario: Adapter backward
When a training step is executed, the adapter gradients MUST update only adapter parameters. THEN the Base Model weights MUST remain unchanged.

#### Scenario: Adapter export and reload
When an adapter is exported to SafeTensors, it MUST include metadata for Base Model identity, architecture, tensor names, dtype, and format version. THEN reloading the artifact MUST restore expected behavior, and an incompatible artifact MUST be rejected without replacing the active adapter.

#### Scenario: Output effect check
When a reloaded adapter is attached, the inference output MUST reflect the adapter's effect. THEN the output difference must be recorded and reproducible.

### Requirement: JNI lifecycle and cargo-ndk build

The Android integration MUST expose a JNI lifecycle with start/stop/status and build via cargo-ndk for ARM64.

#### Scenario: JNI start/stop/status
When the Android Foreground Service starts, the JNI bridge MUST initialize the Rust runtime. THEN stop MUST release resources, and status MUST report a consistent snapshot.

#### Scenario: cargo-ndk ARM64 build
When the Rust library is built with cargo-ndk for Android ARM64, it MUST compile without errors. THEN the resulting `.so` must load in the Android app process.

### Requirement: Checkpoint interruption safety

The checkpoint system MUST preserve the Base Model and previous adapter on interruption.

#### Scenario: Interrupted checkpoint
When a training job is interrupted mid-step, the checkpoint MUST preserve the Base Model weights and the previous adapter. THEN restarting must resume from the latest valid checkpoint without corruption.

### Requirement: Resource measurements

The Phase 0 run MUST record RAM peak, latency, training time, power, temperature, and storage with device-specific thresholds.

#### Scenario: Measurement recording
When the Phase 0 run completes, RAM peak, latency, training time, power, temperature, and storage MUST be recorded. THEN the measurements must include device-specific thresholds for the Poco X7 Pro / Dimensity 8400.

### Requirement: Stop/go criteria

The Phase 0 gate MUST define explicit stop/go criteria.

#### Scenario: Gate pass
When all Phase 0 requirements pass, the gate MUST allow productization. THEN the result must be recorded with evidence.

#### Scenario: Gate fail
When any Phase 0 requirement fails, the gate MUST block productization. THEN the failure must be recorded with root cause.

## Scenarios

#### Scenario: Single path enforcement
When Phase 0 is in progress, the scope MUST remain one model, one backend, one Android device. THEN no multi-model or multi-backend work is permitted.

#### Scenario: No premature claims
When Phase 0 is incomplete, no capability claims for training, Android success, or performance numbers MAY be made. THEN all unverified items remain marked [待驗證] or [待討論].

## Related

- `runtime-core`: owns the model and session objects used in Phase 0.
- `continuous-training`: produces the adapter artifacts validated by the gate.
- `system-guards`: provides the resource measurements required by the gate.