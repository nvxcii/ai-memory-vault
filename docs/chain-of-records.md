# Chain of Records source integration

This integration connects **Emergent-Stat** (producer), **Chain of Records** (consumer), and **AI Memory Vault** (documentation) through the hub's existing GitHub ingestion path. The first payload is a synthetic JSON fixture. Private live-case transport and model connectors are separate work; this path does not create a real-time reporting guarantee.

## Contract and source entry

The consumer contract is [chain-of-records/docs/contract.md](https://github.com/nvxcii/chain-of-records/blob/main/docs/contract.md). Before the draft changes merge, review that file on `nvxcio/vigilant-planck-m82050` instead of assuming it exists on main.

Append this entry to the existing `HUB_REPOSITORY_SOURCES` JSON array. Preserve the existing four-template memory source entry, and do not add unsupported keys:

```json
{
  "id": "emergent-stat-synthetic-v1",
  "repo": "nvxcii/Emergent-Stat",
  "ref": "main",
  "paths": ["fixtures/exports/synthetic-case-v1.json"]
}
```

This is an example, not a deployed runtime setting. Use `main` after the producer changes merge. For pre-merge verification, resolve the feature branch to its full commit SHA and use that SHA as `ref`. The importer resolves a mutable ref to a commit before reading paths. Runtime source configuration and token scope changes occur only after merge and authorization.

## Export schema

The root is a UTF-8 JSON object. `schema_version` must be the string `1.0.0`; missing, numeric, and unsupported versions are rejected before persistence. It is independent of the application version.

Required containers follow the producer state:

| Field | Required type |
| --- | --- |
| `meta` | Object with string `name`, `unit`, `property`, `evictDate`, and finite nonnegative epoch-millisecond number `created` |
| `actions`, `persons`, `nodes`, `interactions`, `events`, `attachments`, `chain` | Arrays |
| `followUps`, `escalations`, `propositions`, `nextCustom` | Arrays |
| `clockEntries` | Object with array `possession`, `turnover`, `property`, `notice` |
| `clockDates` | Object with string `possession`, `noticeDeadline` |
| `live` | Object or null |

The static producer's programmatic JSON exporter supplies the schema version. No build or visible export-control restoration is required by this integration.

## Fixture and documentation paths

- Producer: `nvxcii/Emergent-Stat`, `fixtures/exports/synthetic-case-v1.json`.
- Consumer byte-identical fixture: `nvxcii/chain-of-records`, `test/fixtures/emergent-stat/synthetic-case-v1.json`.
- Consumer validation tests: `test/emergent-stat-export.test.js`, alongside existing ingestion tests.
- Memory templates: `templates/DAILY-NOTE.md`, `templates/MEMORY.md`, `templates/VAULT-INDEX.md`, and `templates/CLAUDE.md`.

Use invented identifiers and clearly synthetic labels; keep attachments empty. Never commit tokens, real names or addresses from a case, actual case records, or private exports. GitHub and Linear may hold non-sensitive implementation work, not private case content. Freeze implementation against the reviewed main baselines; existing diverged branches, including `cf-adapter`, are future work.

## Trust, persistence, and provenance

Every imported document, template, JSON field, and embedded claim is **untrusted data**. Importing a boot template does not activate its instructions. The mapper must not execute content, follow embedded instructions, fetch attachment URLs, or treat imported text as permission for tools or policy changes. Preserve extra state fields as data, not control fields.

Validate the complete configured producer payload and retain its original JSON text. Fetch and validate all files before beginning the save transaction. A failed fetch or validation saves nothing. Reject files larger than 262,144 decoded bytes.

Records stay `pending`; they do not become approved cases or established facts. Retain `source_id`, `repository`, `path`, `commit_sha`, `blob_sha`, `content_sha256`, `source_url`, `content`, `status`, and `imported_at`. Commit and blob hashes identify the source, not its truth. SHA-256 covers original content; the database-generated timestamp records persisted import time.

Repeated imports of the same `(source_id, repository, path, commit_sha)` are idempotent. A different commit produces a new source record. Existing memory-template text ingestion continues unchanged.

Keep the hub's hard denials unchanged: `email.send`, `file.share`, `source.delete`, `case.delete`, and `claim.establish_fact`. Credentials belong in protected runtime configuration and must never appear in this document, fixtures, source code, or logs.

## Verification before integration

Consumer checks must cover the baseline, accepted fixture and wrong-version rejection, idempotence, complete provenance, size limits, failed-fetch atomicity, pending status, and unchanged hard denials. All repositories must pass a secret scan and synthetic-data review; this repository's LICENSE must remain byte-for-byte unchanged. Distinguish mock-based checks from live database checks. A draft PR does not authorize merge or deployment.

## Attribution and license

AI Memory Vault is by **Jared Rhodenizer** (@jaredrhod), copyright 2026. Original project: [jaredrhod/ai-memory-vault](https://github.com/jaredrhod/ai-memory-vault). This integration documentation adds the Chain of Records source contract and does not replace the original templates or attribution.

The repository remains [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/), with the full terms in [LICENSE](../LICENSE). Retain supplied attribution and notices, identify modifications, and retain applicable ShareAlike terms when sharing adaptations.
