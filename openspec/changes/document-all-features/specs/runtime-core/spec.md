# runtime-core

## Purpose

Define model loading, inference session, and training session behavior, and establish the decoupling boundary between model architecture and execution backend.

## ADDED Requirements

### Requirement: Runtime core

The runtime MUST keep model architecture, tokenizer, execution backend, inference, and training capabilities explicitly separated.

#### Scenario: Runtime capability boundary
When the runtime loads a model, it MUST keep model architecture and execution backend as separately declared capabilities.

- The runtime SHALL load a Base Model without requiring a specific tensor backend, so Burn or Candle selection remains a `[待討論]` decision until Phase 0 validates it.
- The runtime MUST maintain an inference session that tracks the active adapter and returns forward results for an input token sequence.
- The runtime MUST maintain a training session that produces adapter weights and supports interruption without corrupting the Base Model.
- The runtime SHALL keep model architecture, tokenizer, and execution backend as separate concerns, so no capability claims any unverified device or backend support.
- The runtime MUST expose backend capability metadata for forward, backward, optimizer, checkpoint, and adapter export/reload operations before a training job starts.
- The tensor/training backend remains `[待討論]`: candidate integrations include LibTorch C++ FFI, a Python subprocess/IPC bridge, or a native Rust backend; none is selected by this specification.
- Weight and adapter exchange MAY use SafeTensors, subject to `[待驗證]` metadata, dtype, architecture, and compatibility rules; GGUF is not assumed to support training.

## Scenarios

#### Scenario: Backend-Independent Model Loading
When the runtime loads a Base Model, it MUST NOT bind to a specific tensor framework. THEN the same load path must work after the Phase 0 vendor is chosen, without rewriting the load logic.

#### Scenario: Inference Session Forward
When an inference session is created for a selected adapter, it SHALL return output tokens for a given input. THEN repeated calls accumulate and return the next token sequence deterministically.

#### Scenario: Interrupted Training Session
When a training session is interrupted mid-step, it MUST preserve the Base Model weights. THEN restarting the session must not corrupt existing adapters.

#### Scenario: Adapter Decoupling
When an adapter is attached to the runtime, it SHALL be addressable by task name. THEN the same Base Model can load different adapters without rebuilding the model weights.

#### Scenario: Backend Capability Negotiation
When a training session is requested, the runtime MUST check the selected backend's declared backward, optimizer, checkpoint, and export/reload capabilities. THEN an unsupported operation fails before mutating model or adapter state.

#### Scenario: Weight Exchange Compatibility
When an adapter is exported for another environment, the artifact MUST include enough metadata to validate Base Model identity, architecture, tensor names, dtype, and format version on reload. THEN incompatible artifacts are rejected without replacing the active adapter.

## Related

- `adapter-routing`: registration and lifecycle of adapters attached to the runtime.
- `inference-api`: exposes the runtime sessions over HTTP/SSE.
