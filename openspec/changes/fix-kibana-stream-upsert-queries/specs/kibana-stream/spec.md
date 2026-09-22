## MODIFIED Requirements

### Requirement: Upsert request shape and stream discriminator (REQ-009)

The resource SHALL build stream upsert requests with the top-level arrays `dashboards` and `rules` always present, even when empty. For Kibana versions before 9.5.0, the request SHALL also always carry a top-level `queries` array, even when empty. For Kibana 9.5.0 and later, and for Serverless, the request SHALL NOT carry a `queries` key, and a configured `queries` attribute SHALL produce an attribute error diagnostic instead of calling the upsert API. The provider SHALL derive the stream type discriminator from the configured block: `wired` for `wired_config`, `classic` for `classic_config`, and `query` for `query_config`. `description` SHALL always be sent from Terraform state.

#### Scenario: Empty dashboards and queries still sent before Kibana 9.5.0

- GIVEN a stream configuration that omits `dashboards` and `queries`
- AND Kibana is older than 9.5.0
- WHEN the provider builds the upsert request
- THEN it SHALL send empty arrays for `dashboards`, `rules`, and `queries` rather than omitting them or sending null

#### Scenario: No queries key on Kibana 9.5.0 and later

- GIVEN a stream configuration that omits `queries`
- AND Kibana is 9.5.0 or later, or Serverless
- WHEN the provider builds the upsert request
- THEN it SHALL send empty arrays for `dashboards` and `rules`
- AND it SHALL NOT send a `queries` key

#### Scenario: Configured queries on Kibana 9.5.0 and later

- GIVEN a stream configuration that sets `queries`
- AND Kibana is 9.5.0 or later, or Serverless
- WHEN create or update runs
- THEN the provider SHALL return an error diagnostic on the `queries` attribute and SHALL NOT call the upsert API

### Requirement: Dashboards and attached queries round-trip through state (REQ-013)

The resource SHALL map `dashboards` to the list of dashboard IDs returned by the Streams API. When the API returns no dashboards, the provider SHALL store `dashboards` as null rather than an empty list. On Kibana versions before 9.5.0, attached `queries` SHALL round-trip `id`, `title`, `description`, `esql`, optional `severity_score`, and optional `evidence`. When the API omits `severity_score`, the provider SHALL store it as null. When the API omits or empties `evidence`, the provider SHALL store it as null. When the API returns no attached queries, the provider SHALL store `queries` as absent.

#### Scenario: Query metadata with optional fields

- GIVEN a stream with an attached query that omits `severity_score` and `evidence`
- WHEN the provider reads the stream into state
- THEN the attached query SHALL have null `severity_score` and null `evidence`
