## Why

Every create and update of `elasticstack_kibana_stream` fails on Kibana 9.5.0 and later, including Elastic Cloud Serverless, with HTTP 400 `{"code":"unrecognized_keys","keys":["queries"],"message":"Excess keys are not allowed","path":[]}`. The provider always sends a top-level `queries` array in `PUT /api/streams/{name}`. elastic/kibana#274128 (first released in 9.5.0) removed `queries` from the upsert schema, and the PUT routes validate strictly. Kibana 9.4 still requires the key.

## What Changes

- Send `queries` in the upsert body only when Kibana is older than 9.5.0. Omit it on 9.5.0+ and Serverless.
- On 9.5.0+ and Serverless, a configured `queries` attribute fails the write with an attribute error, because the resource has no other way to write attached queries.
- Acceptance-test probes send the version-appropriate body. They previously treated the 9.5+ `unrecognized_keys` rejection as "Kibana 9.3 or older" and skipped the whole suite.
- Out of scope: managing significant-event queries through the `/api/streams/{name}/queries` endpoints, which now belong to the separate `significant_events` plugin.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `kibana-stream`: REQ-009 makes the `queries` key version-dependent; REQ-013 limits attached-query round-tripping to Kibana before 9.5.0.

## Impact

- `internal/clients/kibanaoapi/streams.go`: `StreamUpsertRequest.Queries` becomes `*[]StreamQuery` with `omitempty`; new `StreamsUpsertWithoutQueriesMinVersion`.
- `internal/kibana/streams/models.go`, `upsert.go`: version-aware request building.
- `internal/kibana/streams/schema.go`, `docs/resources/kibana_stream.md`: `queries` description states the version limit.
- `internal/kibana/streams/models_test.go`, `acc_test.go`: tests.
