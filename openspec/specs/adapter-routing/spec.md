# adapter-routing Specification

## Purpose
Define Base Model registration, compatibility verification, adapter loading/unloading, versioning, and parallel isolation behavior.

## Requirements

### Requirement: Adapter routing

The adapter router MUST resolve, validate, isolate, version, and safely replace adapters associated with a Base Model.

#### Scenario: Safe adapter route
When a client selects an adapter, the router MUST resolve and validate it before changing active runtime state.

- The router MUST register adapters by task name, such as `edge:notification-classifier`.
- The inference API MUST resolve `base_model:adapter_name` into a Base Model identity and an Adapter identity before loading either selection.
- The router SHALL verify that an adapter is compatible with the active Base Model before attaching it.
- The router MUST prevent parallel inference requests from corrupting the adapter state.
- The router SHALL retain the previous usable adapter version before replacing it.
