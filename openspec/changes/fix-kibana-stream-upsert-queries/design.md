## Context

Kibana 9.4 `baseStreamUpsertRequestSchema` includes `queries: z.array(streamQuerySchema)`, so omitting it is a 400. From 9.5.0 (elastic/kibana#274128) the schema has only `dashboards` and `rules`, and a `queries` key is rejected as unrecognized. The GET response also stops returning `queries`.

## Decisions

- **Gate on version with `EnforceMinVersion(9.5.0-SNAPSHOT)`.** It returns true on Serverless, which runs the newer schema. The threshold is exported from `kibanaoapi` so the resource and the acceptance tests share one value.
- **Pointer field with `omitempty`.** A nil `*[]StreamQuery` omits the key, and a pointer to an empty slice still marshals `[]` for 9.4.
- **Reject configured `queries` on 9.5.0+ rather than drop them.** Dropping them silently would lose configuration, and because GET no longer returns them, every plan would show a diff.
- **Do not call the `/queries` endpoints.** They moved into the `significant_events` plugin, sit behind a feature flag, and have changed shape across recent Kibana versions.

## Risks / Trade-offs

- A practitioner with `queries` configured who upgrades to 9.5.0 now gets an explicit error instead of an opaque 400. The error names the attribute to remove.
