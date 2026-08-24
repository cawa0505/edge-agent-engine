# EdgeAgentDaemon Agent Instructions

## Current State

- This repository is currently documentation-only and is in `Discovery` / Phase 0; there is no Rust manifest, lockfile, CI workflow, test runner, or build command yet.
- Read [`PROGRAM.md`](PROGRAM.md) first for the executable work order and acceptance gate; read [`PROJECT_PLAN.md`](PROJECT_PLAN.md) for architecture and product scope.
- Do not claim model, tensor backend, Android device, performance, thermal, or licensing support until Phase 0 produces reproducible evidence.

## Scope Guardrails

- Phase 0 validates one model, one Rust tensor framework, and one ARM64/Android path; do not build multi-model routing, Dashboard/SaaS, iOS, Vulkan/OpenCL/NPU acceleration, or a full OpenAI-compatible API before the gate passes.
- Do not select Burn vs Candle, the first model, Android binding approach, or Community license speculatively; keep unresolved decisions marked `[待討論]` until verified.
- Keep Base Model, Adapter, training job, checkpoint, and system guard boundaries explicit; a model abstraction must not force inference-only models to implement backward/optimizer behavior.
- Treat battery, temperature, memory, and background-execution thresholds as device-specific policy values, not universal constants.

## Verification

- Any implementation must leave a runnable focused check for non-trivial logic and record reproducible RAM, latency, training-time, power, temperature, checkpoint, and quality measurements where applicable.
- The Phase 0 gate requires real ARM64/Android forward and backward execution, Adapter export/reload, interruption-safe checkpointing, and measured resource usage before productization work begins.
- When build or test configuration is added, document the exact commands here and prefer the repository's executable configuration over plan prose.
