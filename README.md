# AIP Mule System Observability Logs API - Specification

This repository is the contract source of truth for the `aip-mule-sys-observability-logs-api` System API.

## Purpose

Provide a stable, canonical log-retrieval contract across enterprise observability platforms (ELK, Splunk, Datadog, and Azure Monitor) without exposing vendor-specific payloads to downstream consumers.

## Scope

- Define RAML resources, types, examples, and error semantics.
- Protect consumer stability when source platforms evolve.
- Enforce canonical fields for log analysis workflows.

## Source of Truth

- Main contract file: `src/main/resources/api/aip-mule-sys-observability-logs-api.raml`
- Exchange descriptor: `exchange.json`

## Shared Governance Assets

This specification is regenerated from the shared governance baseline and reuses the common fragment asset:

- Template baseline: `mule-utility-template-spec` `1.0.0`
- Common fragment: `mule-utility-common-fragment` `1.0.0`

## Repository Documents

- `docs/contract-overview.md`: architecture intent, compatibility matrix, normalisation strategy, and security controls.
- `docs/changelog.md`: contract evolution log and compatibility notes.

## Ownership and Update Triggers

Update this repository when any of the following occurs:

- Contract path/query/response shape changes.
- New canonical fields are introduced.
- Source adapter capabilities affect consumer-visible behaviour.
- Security controls or data classification handling rules change.

All contract updates require review for compatibility impact and alignment with API contract principles.
