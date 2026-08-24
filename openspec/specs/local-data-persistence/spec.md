# local-data-persistence Specification

## Purpose
Define the local ingestion API, SQLite queue, metadata, file storage, data lifecycle, and sensitive-data handling boundaries.

## Requirements

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
