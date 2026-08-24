# inference-api Specification

## Purpose
Define the local inference interface over HTTP/SSE and Unix Domain Socket, covering the OpenAI-compatible subset of `/v1/models` and `/v1/chat/completions`.

## Requirements

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
