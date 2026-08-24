# system-guards

## Purpose

Define Android/Linux power, idle, temperature, memory monitoring, throttling, pause, and conservative fallback behavior.

## ADDED Requirements

### Requirement: System guards

System guards MUST observe device conditions and prevent unsafe training while exposing an accurate engine state.

#### Scenario: Guarded training
When a configured system guard reports an unsafe condition, training MUST be paused or stopped before further work proceeds.

- The guards MUST observe battery, idle, temperature, and memory conditions on the device.
- Training MUST only proceed when all guard conditions are satisfied.
- When a condition is unsafe, the system SHALL pause or throttle the training job.
- Thresholds MUST be device-specific policy values, not fixed universal constants.
- The engine status model MUST distinguish at least `idle`, `training`, `paused_thermal`, and `paused_oom`.
- The status model MUST expose `thermal_level`, `active_adapter`, and `queue_pending_count` without treating unavailable platform sensors as safe readings.

## Scenarios

#### Scenario: Battery Gate
When the device is on AC power and battery is above threshold, training MAY proceed. THEN when battery drops below threshold, training pauses.

#### Scenario: Thermal Fallback
When temperature exceeds the device policy, the system MUST suspend training. THEN it resumes only after temperature returns below threshold.

#### Scenario: Idle Requirement
When no user input arrives for the required idle window, training SHALL run. THEN any user activity interrupts the training immediately.

#### Scenario: Memory Safety
When available memory falls below the safe level, the system MUST terminate the training job. THEN the Base Model and existing adapters remain intact.

#### Scenario: Status During Thermal Pause
When the thermal guard pauses training, `/v1/system/status` MUST report `paused_thermal` and the observed or unavailable `thermal_level`. THEN it must not report `training` while the job is blocked.

#### Scenario: Status During OOM Protection
When memory protection stops training, `/v1/system/status` MUST report `paused_oom` or the documented terminal OOM state. THEN `queue_pending_count` remains observable and existing model assets remain intact.

## Related

- `continuous-training`: gates whether a training job may start or resume.
- `inference-api`: may throttle inference under thermal or memory pressure.
