# adapter-routing

## Purpose

Define Base Model registration, compatibility verification, adapter loading/unloading, versioning, and parallel isolation behavior.

## ADDED Requirements

### Requirement: Adapter routing

The adapter router MUST resolve, validate, isolate, version, and safely replace adapters associated with a Base Model.

#### Scenario: Safe adapter route
When a client selects an adapter, the router MUST resolve and validate it before changing active runtime state.

- The router MUST register adapters by task name, such as `edge:notification-classifier`.
- The inference API MUST resolve `base_model:adapter_name` into a Base Model identity and an Adapter identity before loading either selection.
- The router SHALL verify that an adapter is compatible with the active Base Model before attaching it.
- The router MUST prevent parallel inference requests from corrupting the adapter state.
- The router SHALL retain the previous usable adapter version before replacing it.

## Scenarios

#### Scenario: Compatibility Check
When an adapter is requested for the active Base Model, the router MUST verify compatibility. THEN an incompatible adapter is rejected before loading.

#### Scenario: Adapter Switch
When a client switches the active adapter, the router MUST preserve the current version. THEN a failed new adapter cannot replace the usable previous one.

#### Scenario: Parallel Isolation
When two inference requests target the same adapter concurrently, each MUST observe a consistent adapter state. THEN neither request sees the other's intermediate tokens.

#### Scenario: Version Retention
When a new adapter replaces the current one, the router SHALL keep the prior version. THEN rollback to the previous adapter must restore its behavior.

#### Scenario: Namespaced Route Rejection
When a request names an unknown adapter or an adapter built for another Base Model, the router MUST reject the route before mutating active state. THEN the previously active Base Model and Adapter remain usable.

## Related

- `runtime-core`: owns the model and session objects the router binds adapters to.
- `continuous-training`: produces adapter versions the router retains.
