# inference-api

## Purpose

Define the local inference interface over HTTP/SSE and Unix Domain Socket, covering the OpenAI-compatible subset of `/v1/models` and `/v1/chat/completions`.

## ADDED Requirements

### Requirement: Local inference API

The local API MUST expose the documented inference, model-listing, and status contracts over HTTP and Unix Domain Socket.

#### Scenario: API availability
When a local client requests a supported inference endpoint, the API MUST return a documented response shape.

- The API MUST expose `/v1/models` returning the list of available Base Models and adapters.
- The API SHALL support `/v1/chat/completions` for both non-streaming and SSE responses.
- The `model` field SHALL accept a base-model namespace in the form `base_model:adapter_name`; the adapter suffix is optional when the base model is requested without an adapter.
- The API MUST expose `GET /v1/system/status` with the current engine state, thermal level, active adapter, and pending ingestion count.
- The Unix Domain Socket MUST accept the same request contract as the HTTP endpoints for local same-machine clients.
- The API MUST document every unsupported field or behavior, rather than claiming full OpenAI compatibility.

## Scenarios

#### Scenario: List Models
When a client requests `/v1/models`, it MUST receive the registered Base Models and adapters. THEN each entry identifies the model name and its current adapter state.

#### Scenario: Streaming Completion
When a client POSTs to `/v1/chat/completions` with streaming enabled, it SHALL receive SSE chunks. THEN the concatenated chunks equal the non-streaming response for the same request.

#### Scenario: Socket Parity
When a local client connects via the Unix Domain Socket, it SHALL receive the same response shape as the HTTP endpoint. THEN a request over the socket returns identical output to the HTTP request.

#### Scenario: Unsupported Field Reporting
When a request includes a field the API does not implement, it MUST respond with a clear error listing the unsupported field. THEN the response must not claim the field is fully supported.

#### Scenario: Namespaced Adapter Selection
When a client submits `model` as `smollm-135m:code-fixer-v1`, the API MUST resolve the base model and adapter as separate namespaced identifiers. THEN an unknown base model or incompatible adapter returns a deterministic client error without changing active runtime state.

#### Scenario: System Status
When a client requests `/v1/system/status`, it MUST receive `status`, `thermal_level`, `active_adapter`, and `queue_pending_count`. THEN the response reflects one consistent engine snapshot and reports an explicit unavailable state when a platform sensor cannot be read.

## Related

- `runtime-core`: provides the inference sessions the API surfaces.
- `system-guards`: gates inference when battery or thermal conditions are unsafe.
