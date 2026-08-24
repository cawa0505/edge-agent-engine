# local-data-persistence

## Purpose

Define the local ingestion API, SQLite queue, metadata, file storage, data lifecycle, and sensitive-data handling boundaries.

## ADDED Requirements

### Requirement: Local data persistence

The local persistence layer MUST durably queue structured training data and keep metadata separate from large model artifacts.

#### Scenario: Durable local record
When a valid training record is submitted, the system MUST persist it in the local queue before acknowledging the request.

- The ingestion API MUST accept training data and enqueue it for later processing.
- `POST /v1/ingest` MUST accept an abstract dynamic payload with `task_id` (string), `messages` (array of `ChatMessage` objects), and `metadata` (object). It MUST NOT require a provider-specific prompt template.
- Each `ChatMessage` MUST identify a role and content; unknown required-field types MUST be rejected before queue insertion.
- The SQLite queue MUST store metadata and ingestion records, not large binary weights.
- Metadata and file storage MUST separate small records from model and checkpoint files.
- Sensitive data handling MUST support retention limits, deletion, and access boundaries.

## Scenarios

#### Scenario: Ingest Data
When a client submits data via `/v1/ingest`, the system MUST enqueue it for training. THEN the data is persisted without immediately executing inference.

#### Scenario: Dynamic Ingest Payload
When a client submits `task_id`, `messages`, and optional `metadata` to `/v1/ingest`, the system MUST preserve the structured payload without applying a fixed prompt template. THEN the queued record can be interpreted by the selected training workflow.

#### Scenario: Invalid Ingest Payload
When `task_id` is missing, `messages` is not an array, or a message has an invalid role/content type, the API MUST reject the request. THEN no partial queue record is created.

#### Scenario: Queue Persistence
When the daemon restarts, records previously enqueued MUST remain available. THEN ingestion order is preserved across restarts.

#### Scenario: Metadata vs File Storage
When a large model file is referenced, the system MUST store the file separately from the SQLite metadata. THEN the metadata entry must not contain the model bytes.

#### Scenario: Data Deletion
When a user requests deletion of sensitive data, the system MUST remove it within the retention boundary. THEN the data is no longer retrievable after deletion.

## Related

- `continuous-training`: consumes the ingestion queue for training jobs.
- `system-guards`: gates ingestion when storage or battery conditions are unsafe.
