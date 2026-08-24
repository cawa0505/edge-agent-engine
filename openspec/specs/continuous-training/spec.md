# continuous-training Specification

## Purpose
Define edge-side adapter fine-tuning workflow behavior: scheduling, interruptible checkpointing, resume, evaluation, and rollback.

## Requirements

### Requirement: Continuous training

The training workflow MUST support guarded, interruptible adapter training with durable checkpoints and safe promotion.

#### Scenario: Interruptible training
When a training job is interrupted between steps, the system MUST retain a valid recovery point and preserve existing model assets.

- The training job MUST support slicing into steps so long runs can be paused and resumed.
- Each step MUST write a checkpoint before proceeding, allowing safe interruption.
- The system SHALL evaluate a new adapter before it replaces the current version.
- A failed adapter MUST not replace the currently usable version.
- The training workflow MUST keep the Base Model and adapter responsibilities separate; inference-only model implementations MUST NOT be forced to provide backward or optimizer behavior.
- Adapter export and reload MUST use a versioned artifact contract; SafeTensors interoperability with Python/PyTorch remains `[待驗證]`.
