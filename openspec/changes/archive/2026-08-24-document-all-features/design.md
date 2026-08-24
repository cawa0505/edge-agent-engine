## Context

EdgeAgent Engine is currently in Discovery phase (Phase 0) with no executable codebase. The project aims to validate a single model on Android/ARM64 using Rust for local inference, adapter training, and resource metrics. The OpenSpec change will formalize core capabilities into specifications without pre-defining models, backends, or Android bindings.

## Goals / Non-Goals

**Goals:**
- Create executable OpenSpec for runtime, inference API, adapter routing, data persistence, training workflow, and system guards
- Define clear boundaries for Phase 0 validation
- Enable future implementation without premature commitments

**Non-Goals:**
- Define specific models (Burn/Candle, Needle/SmolLM)
- Specify Android binding approach (UniFFI/JNI)
- Commit to licensing model (Community/Commercial)
- Include dashboard/SaaS or iOS support

## Decisions

1. **Core Capabilities Scope**
   - *Rationale:* Focus on Phase 0 validation requirements (forward/backward, resource metrics, checkpointing)
   - *Alternatives:* Include model-agnostic patterns prematurely

2. **API Contract Definition**
   - *Rationale:* Limit to OpenAI-compatible subset to avoid over-engineering
   - *Alternatives:* Full API specification

3. **Adapter Versioning**
   - *Rationale:* Version-aware routing prevents breaking changes during training
   - *Alternatives:* Static adapter binding

4. **System Guard Implementation**
   - *Rationale:* Define abstract interfaces before hardware-specific implementations
   - *Alternatives:* Predefine Android-specific guards

## Risks / Trade-offs

[Risk 1: Model Dependency] -> Mitigation: Keep model selection [待討論], validate with single candidate
[Risk 2: Backend Lock-in] -> Mitigation: Document tensor framework [待討論] as optional
[Risk 3: Android Binding] -> Mitigation: Use platform-agnostic interfaces

## Migration Plan

1. Define runtime-core spec (model loading, session management)
2. Specify inference-api contract (HTTP/SSE endpoints)
3. Document adapter-routing logic (versioning, isolation)
4. Outline data persistence requirements (SQLite queue, sensitive data handling)
5. Establish training workflow specifications (checkpointing, rollback)
6. Define system guard interfaces (resource monitoring)

## Open Questions

- [待討論] First model selection (Needle 26M, SmolLM 135M)
- [待討論] Tensor framework (Burn vs Candle)
- [待討論] Android binding approach (UniFFI/JNI)
- 已定案 Community license 採 Apache-2.0

This design artifact focuses solely on documenting capabilities without pre-committing technical choices.