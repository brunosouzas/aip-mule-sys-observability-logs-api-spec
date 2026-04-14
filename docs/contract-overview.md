# Contract Overview - AIP System Observability Logs API

## Context and Intent

This System API provides canonical access to observability logs needed by log-intelligence and AI-assisted diagnosis workflows.

Design intent:

- Keep consumer contracts stable across heterogeneous platforms.
- Prevent Process or Experience APIs from depending on source-specific payloads.
- Align with contract-first, security-by-design, and observability-by-contract principles.

## Source Compatibility Matrix

| Source platform | Native query capability | Canonical mapping coverage | Pagination approach | Known constraints |
| --- | --- | --- | --- | --- |
| ELK (Elasticsearch/Kibana) | Lucene/DSL and time range queries | Full mapping for timestamp, severity, service, environment, message, correlation IDs | Cursor based on search-after token | Field naming often environment-specific; adapter resolves aliases |
| Splunk | SPL with index and sourcetype filters | Full mapping for core fields; partial for trace/span depending on ingestion model | Cursor token generated from latest event position | Query cost can rise with broad ranges; enforce window and limit controls |
| Datadog Logs | Structured search with facets and tags | Full mapping for core fields and tags; trace IDs when present | Cursor based on Datadog page cursor | API rate limits require backoff and partial-results signalling |
| Azure Monitor (Log Analytics) | KQL query execution | Full mapping for core fields; partial for custom dimensions | Continuation token from query results | Workspace schema variance across tenants; adapter applies table profile |

## Unified Contract Draft (RAML)

Reference: `src/main/resources/api/aip-mule-sys-observability-logs-api.raml`

Template and fragment alignment:

- Reuses `mule-utility-common-fragment` via `uses` (`core-lib` alias).
- Applies shared traits:
  - `core-lib.pageable`
- Reuses shared health-check resource types:
  - `core-lib.healthCheckAlive`
  - `core-lib.healthCheckReady`
- Uses shared client-id enforcement scheme through `core-lib.token`.

Key operations:

- `GET /logs`: canonical search endpoint with stable query filters and paginated response.
- `GET /sources/health`: operational visibility into source adapter state.
- `GET /health-check/alive` and `GET /health-check/ready`: shared platform health conventions.

Canonical response guarantees:

- `entries[]` always conforms to the same `LogEntry` schema.
- `sourcePlatform` identifies origin, but vendor payload shape is never exposed.
- `dataClass` and `redactionApplied` are explicit on each record.
- Error and health-check conventions stay aligned through shared fragment assets and reusable resource types.

## Canonical Normalisation Strategy

1. Ingest source payload with adapter-specific parser.
2. Map source fields to canonical model:
   - Event identifier -> `logId`
   - Event time -> `timestamp`
   - Severity level -> `severity` (TRACE/DEBUG/INFO/WARN/ERROR/FATAL)
   - Service/application identifier -> `serviceName`
   - Environment/tag -> `environment`
   - Message/body -> `message`
   - Correlation/trace/span fields -> `correlationId`, `traceId`, `spanId`
3. Classify data by policy (`public`, `internal`, `confidential`, `restricted`).
4. Apply redaction/tokenisation before response assembly.
5. Emit canonical object only, with optional generic `attributes` map for safe non-core metadata.

Normalisation rules:

- Prefer deterministic mapping tables per source and tenant profile.
- Unknown severity values are translated via policy map (for example, `notice` -> `INFO`).
- Missing optional IDs remain null/absent; contract does not fail when non-mandatory telemetry is unavailable.
- If one source fails, return partial results with source health degraded indicators.

## Security and Compliance Controls

Credential and access controls:

- Source credentials stored in enterprise secrets manager; never in code or properties files.
- Client ID enforcement (shared `core-lib.token` scheme) restricts authorised access.
- Least-privilege connector service accounts per source tenant/workspace.

Data protection controls:

- Restricted fields (tokens, secrets, personal identifiers) masked before storage and before response.
- Data classification performed before any enrichment or downstream publication.
- Restricted data is prohibited from external LLM egress paths unless explicitly sanitised and approved.

Operational controls:

- `correlation_id` required for traceability across adapters and API layers.
- Immutable audit trail for query actor, source selection, and policy decisions.
- Rate limiting and query-window controls to reduce abuse and cost spikes.
- Retention and deletion policies applied according to data classification and legal requirements.

## Non-Functional Considerations

- **Reliability**: adapter timeout isolation, circuit breaker per source, partial-results response policy.
- **Observability**: metrics for adapter latency/error rate, request volume, redaction count, and partial-results ratio.
- **Cost**: query-window enforcement, default limits, and optional source filter to prevent unnecessary multi-source fan-out.

## Review and Evolution

Update this document when:

- RAML resources or schema fields change.
- A new source adapter is added or mapping policy changes.
- Security/compliance rules are revised.

Breaking contract changes require explicit architectural approval and versioning impact assessment.
