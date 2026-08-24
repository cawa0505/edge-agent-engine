# continuous-training

## Purpose

Define edge-side adapter fine-tuning workflow behavior: scheduling, interruptible checkpointing, resume, evaluation, and rollback.

## ADDED Requirements

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

## Scenarios

#### Scenario: Interrupted Training
When a training job is interrupted, it MUST retain the latest checkpoint. THEN resuming from that checkpoint continues the same job without restarting from scratch.

#### Scenario: Failed Adapter Rollback
When an adapter fails evaluation, the system MUST keep the prior version. THEN the previous adapter remains active and usable.

#### Scenario: Scheduled Job
When a training job is scheduled, it SHALL run only when system guard conditions are met. THEN it does not start when battery or thermal thresholds are unsafe.

#### Scenario: Checkpoint Integrity
When a training step completes, it MUST write the checkpoint atomically. THEN a crash mid-step must not leave a corrupt checkpoint.

#### Scenario: Backend Without Training Capability
When the selected runtime backend cannot provide the required backward or optimizer capability, the training job MUST remain unstarted. THEN inference capability and existing adapters remain available.

#### Scenario: Adapter Export and Reload
When a completed adapter is exported, the system MUST persist version and compatibility metadata with the artifact. THEN reload either restores the adapter or returns a compatibility error without changing the active version.

## Related

- `adapter-routing`: manages adapter versions produced by training.
- `system-guards`: gates training start and resume on resource conditions.
