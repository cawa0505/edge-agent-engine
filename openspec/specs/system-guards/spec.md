# system-guards Specification

## Purpose
Define Android/Linux power, idle, temperature, memory monitoring, throttling, pause, and conservative fallback behavior.

## Requirements

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
