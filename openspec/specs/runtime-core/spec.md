# runtime-core Specification

## Purpose
Define model loading, inference session, and training session behavior, and establish the decoupling boundary between model architecture and execution backend.

## Requirements

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
