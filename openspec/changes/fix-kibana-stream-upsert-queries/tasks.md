## 1. Tests

- [x] 1.1 Unit tests in `internal/kibana/streams/models_test.go` marshal the upsert body and assert the `queries` key for both request shapes, with and without configured queries.

## 2. Fix

- [x] 2.1 Make `StreamUpsertRequest.Queries` an omitempty pointer and add `StreamsUpsertWithoutQueriesMinVersion`.
- [x] 2.2 Pass the version decision from `writeStream` into `toAPIUpsertRequest`; add the attribute error for configured queries on 9.5.0+.
- [x] 2.3 Make acceptance-test probes and the attached-query step version-aware.

## 3. Docs and spec

- [x] 3.1 Update the `queries` schema description and `docs/resources/kibana_stream.md`.
- [x] 3.2 Sync the delta into `openspec/specs/kibana-stream/spec.md`.
