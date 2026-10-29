# OpenKPIs catalog field reference v3

Status: canonical contract for the production database, site, GitHub content, and CGPT workflow. The coordinated schema and application cutover was completed on October 26, 2026. A headed-Chrome 1/1/1/1 workflow-quality pilot remains a separate operational gate.

## Contract outcome

- 44 stored content fields exist across KPI, Metric, Dimension, and Event records.
- `entity_type` is a virtual workflow field, producing 45 authoring fields in the contract.
- Applicability including `entity_type`: KPI 34, Metric 35, Dimension 35, Event 37.
- Every catalog table also carries 18 workflow/provenance fields that agents do not author as catalog content.
- The Add form remains minimal. The full contract belongs in Edit, public detail pages, published GitHub item files, and the field-reference documentation.
- `core_area` means Journey / Navigation Area. `owner` means Owning Function. They are not interchangeable.
- `priority` is directly renamed to `business_criticality`; there is no permanent dual-read or dual-write compatibility layer.
- The mixed `event_type` field is replaced by `ga4_event_type` and `adobe_event_type`.
- Events do not use `aggregation_window`. Their counting and duplicate behavior belongs in `event_serialization`.
- Dimensions may use `aggregation_window`, displayed as **Lookback Window**, only when historical behavior is needed to determine the Dimension value.

## Surface rules

**Add form** remains unchanged and minimal:

- All types: `name`, `slug`, `description`, `category`, `tags`.
- KPI, Metric, Dimension: `formula`.
- Event: `event_serialization`.

`slug` is editable only while creating the record. After creation it is a stable, read-only identity. Every other applicable content field is available in Edit. Public pages show all meaningful applicable fields, hiding empty or inapplicable technical sections. Published GitHub item files include all applicable content fields and safe provenance. Internal UUIDs, email addresses, queue state, and sync internals are never published.

## Agent field protocol

Every field definition supplied to an agent must include:

1. `purpose`: what the field represents and which reader it helps.
2. `expected_value`: exact content and quality standard.
3. `storage`: database type and applicability.
4. `surface`: Add, Edit, Public, and GitHub behavior.
5. `reads_from`: already-approved fields the agent must consume.
6. `feeds`: later fields that must reuse this value.
7. `invariants`: cross-field rules that cause rejection when broken.
8. `limits`: length, item-count, or structural limits.
9. `good_examples` and `bad_examples`.

An agent may author only its assigned modular field group. It receives the immutable approved output of earlier groups, returns a patch for its own fields, and may not silently rewrite upstream fields. A later field must reuse the upstream value rather than independently inventing an alternative.

## 45-field authoring contract

### Identity, discovery, and business meaning

| # | Field | Applies | DB type | UI / GitHub | Definition and authoring prompt | Reads from → feeds |
|---:|---|---|---|---|---|---|
| 0 | `entity_type` | K/M/D/E | Virtual; table/path determines type | Workflow only; include as `Type` in GitHub | Choose exactly `kpi`, `metric`, `dimension`, or `event`. KPI is decision-oriented against an objective; Metric is a reusable quantity; Dimension is a descriptive breakdown; Event is one timestamped occurrence. | Request → applicable schema and all later fields |
| 1 | `name` | K/M/D/E | `text NOT NULL` | Add/Edit/Public/GitHub | Established general-reader term only; 3–100 characters, normally at most 10 words. No definition, duration, platform, or implementation jargon unless integral to the concept. | `entity_type` → `slug`, `aliases`, `description`, Event names |
| 2 | `slug` | K/M/D/E | `text NOT NULL` | Add; read-only afterward; URL/GitHub | Stable lowercase kebab-case identity, 3–120 characters, normally derived from `name`. Do not change after publication except through an explicit redirect migration. | `name` → URLs, GitHub path, dependencies and relationships |
| 3 | `aliases` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | Zero to eight established alternate names or abbreviations for the same concept. Exclude dependencies, related concepts, platform event tokens, and speculative variants. | `name` → search and display |
| 4 | `description` | K/M/D/E | `text` | Add/Edit/Public/GitHub | One or two plain-language sentences, 40–220 characters, defining what the item means and why it matters. Do not include the formula, fixed defaults, data-layer instructions, or tracking-plan prose. | `name`, `entity_type` → formula, use case, QA |
| 5 | `category` | K/M/D/E | `text` | Add/Edit/Public/GitHub | One primary measurement-purpose value: Acquisition, Engagement, Conversion, Revenue & Monetization, Retention, Satisfaction, Operational Efficiency, Technical Reliability, Data Quality & Governance, or Other. For KPI, Metric, and Event records, classify what the record measures or represents: a purchase Event or purchase-specific identifier is Conversion; an amount, price, quantity, currency, basket-size field, or confirmed-order count is Revenue & Monetization. A preference/method selection that does not complete a declared objective is Engagement. Operational Efficiency requires process execution such as throughput time, cost, capacity use, accuracy, or completion. For Dimensions, classify the primary analysis domain represented; Fulfillment Channel and Inventory Availability Status are Operational Efficiency. Do not let owner or use case override the record meaning. | `description` and record meaning → navigation, discovery |
| 6 | `tags` | K/M/D/E | `text[]` | Add/Edit/Public/GitHub | Two to eight distinct short search labels, each at most 40 characters. Include domain/journey terms; avoid sentences, aliases already captured, or stuffing. | `name`, `category`, `core_area` → search |
| 7 | `industry` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | Every industry where the unchanged definition genuinely applies. Site taxonomy: Retail, E-commerce, SaaS, Healthcare, Education, Finance, Media, Technology, Manufacturing, Travel & Hospitality, Telecommunications, Automotive, Other. E-commerce/Retail is the current generation focus, not a taxonomy restriction. | `description` → discovery and portfolio selection |
| 8 | `business_criticality` | K/M/D/E | `text` with UI validation | Edit/Public/GitHub; not Add | `Critical` for regulatory, financial, customer-risk, or core-operational decisions where wrong/missing data creates material harm; `High` for routine material decisions; `Medium` for optimization/management; `Low` for diagnostic/specialized support. Never infer from whether the current measured result is good or bad. | `description`, `business_use_case`, `dependencies` → governance and review depth |
| 9 | `core_area` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | One to three Journey / Navigation Areas where the concept occurs: Acquisition, Discovery, Consideration, Product Experience, Cart, Checkout, Payment, Purchase, Post-Purchase, Fulfillment, Returns & Refunds, Loyalty & Retention, Customer Service, Account & Identity, Store Experience, Cross-Journey. This is not the owning function. | `description` → navigation and tags |
| 10 | `owner` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | The single Owning Function accountable for the definition, quality, and business interpretation: Marketing, Merchandising, Product Management, Digital Experience, Customer Experience, Customer Support, Store Operations, E-commerce Operations, Order Management, Fulfillment & Logistics, Inventory, Payments, Returns, Loyalty & CRM, Finance, Risk & Compliance, Data & Technology, or People Operations. This is not the contributor. | `description`, `business_use_case` → provenance and governance routing |
| 11 | `scope` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | Closest standard measurement level: User, Session, Event, or Product. Purchases, orders, carts, payments, shipments, returns, and other transaction occurrences use Event; items use Product. Preserve the precise business grain in the definition, formula, dependencies, parameters, and SQL. | `description` → formula, parameters, mappings, SQL |

### Collection context, lineage, reporting, privacy

| # | Field | Applies | DB type | UI / GitHub | Definition and authoring prompt | Reads from → feeds |
|---:|---|---|---|---|---|---|
| 12 | `measurement_channels` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | Select where the measured action or fact occurs: Web, Mobile App, Backend / API, Store / POS, Marketing Messaging, Call Center / Support, External Platform / Partner, Connected Device / IoT, or Other Offline. External Platform / Partner covers third-party sites, marketplaces, social-platform forms or commerce, and affiliate/partner-hosted experiences. Paid, organic, referral, affiliate, and social acquisition classifications remain Report Attributes/Dimensions when the action itself occurs on Web or Mobile App. | `description`, `scope` → technical applicability, source data, mappings |
| 13 | `source_data` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | One to five vendor-neutral source families: Digital Analytics, Commerce Platform, Order Management, Product Information, Point of Sale, Inventory or Warehouse, CRM or CDP, Marketing Platform, Customer Support, Payment Processor, Finance or ERP, Experimentation Platform, Survey or Voice of Customer, Application or Server Logs, Offline Operational System, External Partner Data. | `measurement_channels`, `dependencies` → mappings and SQL applicability |
| 14 | `dependencies` | K/M/D/E | `jsonb` | Edit/Public/GitHub; not Add | Exact object with `Events`, `Metrics`, `Dimensions`, and `KPIs` arrays of published canonical slugs. Include only direct upstream inputs that must be captured/created first. Exclude aliases, optional slices, topical peers, downstream outputs, platform event names, prose, self-reference, and cycles. | `formula` or Event contract, catalog lookup → event inventories, mappings, SQL |
| 15 | `report_attributes` | K/M/D/E | `text[]` | Edit/Public/GitHub; not Add | One to eight available Dimensions that business users can filter, group, or compare by. Use recognizable names/slugs; exclude measures, implementation keys, unavailable fields, and required calculation inputs already in dependencies. | `scope`, `dependencies`, available Dimensions → dashboards and SQL projection |
| 16 | `dashboard_usage` | K/M | `text[]` | KPI/Metric Edit/Public/GitHub; not Add; hidden for D/E | Optional zero to three generic **Recommended Reporting Views**, 2–6 words and at most 60 characters each. Derive them from the business use case, owner, category, and available report attributes. Do not claim a deployed dashboard or invent an organization-specific name. | `business_use_case`, `owner`, `category`, `report_attributes` → public recommendation |
| 17 | `segment_eligibility` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | One business-readable audience condition, 8–220 characters. Use uppercase operators `EQUALS`, `NOT EQUALS`, `GREATER THAN`, `GREATER THAN OR EQUAL TO`, `LESS THAN`, `LESS THAN OR EQUAL TO`, `IN`, `NOT IN`, `CONTAINS`, `NOT CONTAINS`, `STARTS WITH`, `ENDS WITH`, `EXISTS`, `NOT EXISTS`, `BETWEEN`, `AND`, `OR`. Use `NOT APPLICABLE — reason` when segmentation is meaningless. Do not write Yes/No, SQL, `LIKE`, `WHERE`, or `THEN`. | `scope`, `data_type`, `parameters`, `pii_flag` → audience guidance and privacy QA |
| 18 | `data_sensitivity` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | Public, Internal, or Restricted based on the most sensitive required input, identifier, or output—not on whether the catalog page is public. | `dependencies`, `parameters`, mappings → access and QA |
| 19 | `pii_flag` | K/M/D/E | `boolean` | Edit/Public/GitHub; not Add | `true` only when the implementation requires direct or linkable personal identifiers; otherwise `false`. It must agree with parameters, identityMap, mappings, SQL, and Data Sensitivity. Do not assume every anonymous ID is PII or ignore joinable identity. | `parameters`, `xdm_mapping`, SQL identifiers → privacy QA |

### Business calculation and type-specific meaning

| # | Field | Applies | DB type | UI / GitHub | Definition and authoring prompt | Reads from → feeds |
|---:|---|---|---|---|---|---|
| 20 | `formula` | K/M/D | `text` | Add/Edit/Public/GitHub | Shortest unambiguous business expression, 5–300 characters. Use recognizable quantities: `(Orders from previously abandoned carts / Total abandoned carts) × 100`. Move timing, eligibility, exclusions, deduplication, attribution, and zero-denominator handling to Calculation Notes unless essential to the identity. For Dimensions, use only when a transformation/derivation is required. | `name`, `description`, `scope` → measure type, dependencies, notes, SQL |
| 21 | `measure_type` | K/M | `text` | Edit/Public/GitHub; not Add | Exactly one mathematical type matching the formula and display unit: Counter, Rate, Ratio, Percentage, Average, Sum, or Currency. A value multiplied by 100 and shown with `%` is Percentage, not Rate. | `formula` → SQL unit QA |
| 22 | `data_type` | D | `text` | Edit/Public/GitHub; not Add | Actual returned value type: string, number, boolean, date, datetime, or list. Identifiers/codes are strings. JSON, counter, array, KPI, Metric, and Event are not Dimension data types. | `description`, `formula` → parameters, segment condition, SQL |
| 23 | `aggregation_window` | K/M/D | `text[]` | Edit/Public/GitHub; not Add; label Dimension as **Lookback Window** | KPI/Metric: supported reporting windows. Dimension: optional evaluation/lookback window only when prior behavior determines the value. Use Event, Session, Selected Reporting Period, Hourly, Daily, Weekly, Monthly, Quarterly, Yearly, Rolling 7 Days, Rolling 30 Days, Rolling 90 Days, or Configured Lookback. A fixed duration is allowed only when integral to the named definition; never invent a universal default. | `formula`, `scope`, `calculation_notes` → SQL |
| 24 | `calculation_notes` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | One to five short numbered rules covering only necessary timing, eligibility, exclusions, deduplication, attribution, identity stitching, or zero-denominator handling. One rule per bullet, normally at most 120 characters. No filler or implementation history. | definition, formula, windows, Event rules → mappings, SQL, QA |
| 25 | `business_use_case` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | One to three short numbered uses. Each names the actor, the signal/trigger, and the decision or action. Do not repeat the description or say only that the item is important. | `description`, `owner`, `business_criticality` → dashboard usage and public explanation |

### Canonical event crosswalk and collection mappings

| # | Field | Applies | DB type | UI / GitHub | Definition and authoring prompt | Reads from → feeds |
|---:|---|---|---|---|---|---|
| 26 | `event_serialization` | E | `text` | Add/Edit/Public/GitHub | One concise firing/counting/deduplication rule, 3–120 characters, including the stable key where applicable. Example: “Fire once after order confirmation; deduplicate by `transaction_id`.” It is not a JSON serialization format. | `scope`, `trigger_condition` → parameters, mappings, calculation notes |
| 27 | `trigger_condition` | E | `text` | Edit/Public/GitHub; not Add | One business condition, 12–180 characters, in `WHEN …, THEN emit …` form. Use the completed action or state transition, not a click that merely precedes it. | `name`, `scope` → event names, serialization, mappings |
| 28 | `ga4_event_type` | E | `text` | Edit/Public/GitHub; not Add | Independently classify the GA4 event as Automatically Collected, Enhanced Measurement, Recommended, Custom, or Not Applicable using the dated official-source local registry. Hold context-dependent tokens such as `ad_impression` until the collection method is known. | `ga4_event`, channels, GA4 registry → GA4 validation |
| 29 | `adobe_event_type` | E | `text` | Edit/Public/GitHub; not Add | Independently classify the Adobe XDM `eventType` as Standard XDM, Custom XDM, or Not Applicable using the local Adobe registry. Do not mix this with Analytics commerce events or implementation-defined ACDL names. | `adobe_event`, channels, Adobe registries → XDM validation |
| 30 | `ga4_event` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | Event record: one lower_snake_case GA4 token selected from the official-source registry when a standard token fits; otherwise a valid Custom token. KPI/Metric/Dimension: one `event_name — short purpose` line for every required Event dependency. Derived records copy names from `dependencies.Events`. Offline-only records leave this empty. | Event: `name`, `trigger_condition`, GA4 registry; derived: `dependencies.Events` → GA4 Data Layer, parameters, BigQuery SQL |
| 31 | `adobe_event` | K/M/D/E | `text` | Edit/Public/GitHub; not Add | Event record: one current standard or custom XDM `eventType`, never deprecated tokens or `event12`-style Analytics IDs. KPI/Metric/Dimension: one `event_name — short purpose` line per required Event dependency. Reuse the canonical custom event name across platforms when both are custom. Offline-only records leave this empty. | Event: `name`, `trigger_condition`, Adobe registry; derived: `dependencies.Events` → Adobe layer, XDM, parameters, AEP SQL |
| 32 | `parameters` | E | `jsonb` | Edit/Public/GitHub; not Add | Compact object with at most 20 lower_snake_case keys. Each has `type`, `required`, `meaning` (≤80 characters), and digital `paths`. Resolve standard fields from the local RAG. Each custom field also has one or more controlled `purposes` and a 12–120 character necessity `justification`. A scalar total may use repeatable line paths only with an explicit sum and identical amount basis. GA4 purchase/refund value excludes tax/shipping and aligns to summed XDM item price/refund amounts, not `commerce.order.priceTotal`. | `scope`, trigger, serialization, platform names and local RAG → all mappings and SQL |
| 33 | `w3_data_layer` | K/M/D/E | `jsonb` | Edit/Public/GitHub; not Add | Valid CEDDL JSON rooted at `digitalData`, using registered standard paths or documented `attributes` extension points. `digitalData.event` contains exactly the canonical Event set. Hold an unknown path; never invent one. | Event dependencies, parameters and W3 registry → GA4/Adobe mappings and parity QA |
| 34 | `ga4_data_layer` | K/M/D/E | `jsonb` | Edit/Public/GitHub; not Add | JSON array with one payload per GA4 Event. Use the Event-specific registry: include required/conditional parameters, item_id or item_name for each item, documented ecommerce fields under `ecommerce`, and justified custom parameters as root siblings. | GA4 Event, parameters and GA4 registry → BigQuery SQL |
| 35 | `adobe_client_data_layer` | K/M/D/E | `jsonb` | Edit/Public/GitHub; not Add | JSON array with one pushed object per Adobe Event. Business fields must use `data`, `eventInfo`, or an approved XDM-shaped envelope; arbitrary business keys at the message root are not allowed. | Adobe Event, parameters and ACDL registry → XDM and AEP SQL |
| 36 | `xdm_mapping` | K/M/D/E | `jsonb` | Edit/Public/GitHub; not Add | JSON array with one ExperienceEvent per Adobe Event. Each has `_id`, `timestamp`, matching `eventType`, and paths whose types resolve to the local XDM registry or an explicitly configured tenant namespace. | Adobe Event, parameters, ACDL and XDM registry → AEP SQL and PII QA |

### SQL and catalog relationships

| # | Field | Applies | DB type | UI / GitHub | Definition and authoring prompt | Reads from → feeds |
|---:|---|---|---|---|---|---|
| 37 | `ga4_sql` | K/M/D | `text` | Edit/Public/GitHub; not Add; absent for Event | One concise read-only BigQuery query built after the Event/parameter crosswalk. Translate collection paths to export fields: `value`, `currency`, and custom keys come from `event_params`; item keys come from `items`; never query collection-only `ecommerce.value`, `ecommerce.currency`, or `ecommerce.items`. Use every and only declared GA4 Events and the smallest mapped parameter subset required by this record. Correlate progressions by the exact shared identifier and require the outcome after the start. A descriptive Dimension returns the collected value unchanged unless its definition requires a transformation. Bound inputs before deduplication/conflict checks so post-period data cannot change history. Do not invent duplicate-conflict policies; keep missing source rules as integration prerequisites. Treat `_TABLE_SUFFIX` as coarse pruning in the property reporting timezone; buffer UTC boundaries when the property timezone is not fixed and enforce exact bounds with `event_timestamp`. Prefer ≤25 lines; simple one-event sums and Dimensions use a direct query or at most one CTE and one join; block above the earned 30/50/80-line tier; never use `SELECT *`. | definition, formula, scope, notes, GA4 inventory/parameters and BigQuery registry → deterministic QA |
| 38 | `aep_sql` | K/M/D | `text` | Edit/Public/GitHub; not Add; absent for Event | One concise read-only AEP query from the same approved Event inventory and minimal item-specific parameter subset as GA4 SQL, using the selected dataset profile and only mapped XDM paths. Select one canonical identity from the configured `identityMap` namespace per event; never explode aliases into extra entities. Apply the same progression correlation, unchanged Dimension projection, bounded-input, and no-invented-conflict-policy rules as GA4 SQL. Preserve identical grain, rules, and output. Use the same 25 preferred and 30/50/80 maximum tiers. | definition, formula, scope, notes, Adobe inventory/XDM/AEP registry → cross-platform QA |
| 39 | `related_kpis` | K | `text[]` | Edit/Public/GitHub; not Add | Zero to eight existing KPI slugs useful for comparison or interpretation. Exclude prerequisites, aliases, cross-type records, and duplicates. | catalog search, dependencies → discovery |
| 40 | `related_metrics` | M | `text[]` | Edit/Public/GitHub; not Add | Zero to eight existing Metric slugs useful for comparison or interpretation. Exclude prerequisites, aliases, downstream KPIs, and duplicates. | catalog search, dependencies → discovery |
| 41 | `related_dimensions` | D/E | `text[]` | Edit/Public/GitHub; not Add | Zero to eight existing Dimension slugs providing adjacent context but not directly extracted/derived from this record. | catalog search, dependencies → discovery |
| 42 | `derived_kpis` | M/E | `text[]` | Edit/Public/GitHub; not Add | Existing KPI slugs whose calculation directly consumes this Metric/Event. Topical similarity is insufficient. | catalog reverse-lineage check → discovery and lineage QA |
| 43 | `derived_metrics` | E | `text[]` | Edit/Public/GitHub; not Add | Existing Metric slugs whose calculation directly consumes this Event. | catalog reverse-lineage check → discovery and lineage QA |
| 44 | `derived_dimensions` | D/E | `text[]` | Edit/Public/GitHub; not Add | Existing Dimension slugs directly transformed or extracted from this Dimension/Event. Independently collected peers belong in `related_dimensions`. | catalog reverse-lineage check → discovery and lineage QA |

## Required cross-field invariants

These are publication blockers, not writing suggestions:

1. `name` is the concept; `aliases` are equivalent names; `related_*` are optional peers; `dependencies` are required upstream inputs; `derived_*` are direct downstream outputs.
2. `core_area` is journey/navigation placement. `owner` is the accountable operating function. `category` is measurement purpose. `industry` is applicability. `scope` is grain.
3. The formula, description, measure type, scope, aggregation/lookback window, and calculation notes describe one definition with no hidden fixed default.
4. Every `dependencies.Events` slug resolves to an Event record. The derived record's GA4 and Adobe inventories are copied from those Event records.
5. `ga4_event` names equal the root `event` values in `ga4_data_layer` and the `event_name` filters in `ga4_sql`.
6. `adobe_event` names equal the pushed event values in `adobe_client_data_layer`, the `eventType` values in `xdm_mapping`, and the filters in `aep_sql`.
7. When both platform events are custom, the same canonical lower_snake_case business name is used unless authoritative platform constraints require a documented exception.
8. Event `parameters` is the parameter registry. Every business field in W3, GA4, Adobe, XDM, or SQL appears there; every declared path exists in the relevant payload.
9. GA4 and AEP SQL implement the same formula, grain, eligibility, timing, exclusions, deduplication, and null/zero behavior.
10. Digital/hybrid records require applicable platform events, mappings, and SQL. Offline-only records do not fabricate browser artifacts. `measurement_channels` and `source_data` determine this branch.
11. `pii_flag` and `data_sensitivity` agree with identifiers in parameters, identityMap, mappings, and SQL.
12. No agent may approve its own output. Deterministic gates run before adversarial review, and failed fields return to the owning module with the immutable upstream contract.

## Modular authoring sequence

1. **Identity module:** `entity_type`, `name`, `slug`, `aliases`.
2. **Business-definition module:** `description`, `formula`, `category`, `scope`, `measure_type`/`data_type`, `aggregation_window`, `calculation_notes`, `business_use_case`.
3. **Classification and governance module:** `tags`, `industry`, `business_criticality`, `core_area`, `owner`, `dashboard_usage`.
4. **Lineage and privacy module:** `measurement_channels`, `source_data`, `dependencies`, `report_attributes`, `segment_eligibility`, `data_sensitivity`, `pii_flag`, related/derived fields.
5. **Event-contract module:** for Events, `trigger_condition`, `event_serialization`, `ga4_event_type`, `adobe_event_type`, `ga4_event`, `adobe_event`, `parameters`. For K/M/D, it reads Event dependencies and builds the platform inventories without renaming them.
6. **Mapping module:** `w3_data_layer` first, followed by `ga4_data_layer`, `adobe_client_data_layer`, and `xdm_mapping` from the same event/parameter registry.
7. **SQL module:** `ga4_sql` and `aep_sql` from the approved business definition and platform mappings.
8. **Independent QA:** schema validation, cross-field parity, lineage, plain-language quality, compactness, privacy, SQL equivalence, public rendering, UI read-back, GitHub sync, and public-page verification.

## 18 workflow and provenance fields

These are stored in each K/M/D/E table but are not AI-authored catalog content.

| Field | Target type | Set by | Public/GitHub behavior |
|---|---|---|---|
| `id` | `uuid` | Database | Internal only |
| `status` | `text` | Workflow | GitHub may show Published; public pages do not need a redundant badge |
| `validation_status` | `text` | Deterministic/QA workflow | Public provenance may show Passed; internal failure detail stays internal |
| `version` | `integer` | Publication workflow | Public and GitHub |
| `created_by` | `uuid` | Auth/workflow | Never expose UUID/email; resolve to safe display name |
| `created_at` | `timestamptz` | Database | Public and GitHub date |
| `last_modified_by` | `uuid` | Auth/workflow | Resolve to safe display name only |
| `last_modified_at` | `timestamptz` | Workflow | Public and GitHub date |
| `reviewed_by` | `uuid[]` | Governance | Resolve to safe display names if shown |
| `reviewed_at` | `timestamptz` | Governance | Public provenance when present |
| `approved_by` | `uuid` | Governance | Resolve to safe display name only |
| `approved_at` | `timestamptz` | Governance | Public and GitHub when present |
| `publisher_id` | `uuid` | Publication workflow | Internal ID; display safe publisher name only |
| `published_at` | `timestamptz` | Publication workflow | Public and GitHub |
| `github_pr_url` | `text` | GitHub sync | Editor/admin; optional public source history |
| `github_pr_number` | `integer` | GitHub sync | Internal when URL is present |
| `github_commit_sha` | `text` | GitHub sync | Public Source link and GitHub provenance |
| `github_file_path` | `text` | GitHub sync | Public Source link; internal raw path need not be displayed |

Public provenance displays Created by/at, Last updated by/at, Reviewed/Approved/Published by/at when present, Version, Validation state, and GitHub Source. It never displays email addresses, authentication IDs, or contact information.

## Target database field counts

Counts exclude legacy columns after their data is migrated and verified.

| Table | Content fields | Workflow/provenance | Final stored columns |
|---|---:|---:|---:|
| KPI | 33 | 18 | 51 |
| Metric | 34 | 18 | 52 |
| Dimension | 34 | 18 | 52 |
| Event | 36 | 18 | 54 |

`entity_type` is not stored because the table and route already encode it.

## Legacy migration decisions

No legacy column is dropped until its populated values are classified, migrated, counted, and verified.

| Legacy field | Final action |
|---|---|
| `priority` | Direct column rename to `business_criticality`; coordinated maintenance window; no dual field |
| `event_type` | Split each Event into `ga4_event_type` and `adobe_event_type`, validate, then remove |
| `sql_query` | Classify and migrate each populated value to `ga4_sql` or `aep_sql`; archive ambiguous values for review |
| `kpi_type`, `metric_type` | Migrate genuine values to `measure_type`, then remove |
| `measure_aggregation` | Migrate any genuine value to `aggregation_window`, then remove |
| `dashboards` | Migrate to `dashboard_usage`, then remove |
| `related_events` | Migrate required inputs to `dependencies.Events`; optional peers are not retained as dependencies |
| `github_author` | Preserve contribution provenance through user/profile metadata; remove duplicate column |
| `amplitude_implementation` | Remove after confirming zero values and removing code fallback |
| `raw_json` | Archive the few populated blobs outside the live catalog, then remove |
| `search_vector` | Remove only if search does not use it; otherwise rebuild as an internal generated/search column, not content |
| KPI `related_dimensions`, `related_metrics` | Remove after confirming no useful data; KPI prerequisites belong in `dependencies` |
| Metric `related_kpis` | Reclassify populated values to `derived_kpis`, `dependencies`, or `aliases`, then remove |
| Event `aggregation_window` | Remove; migrate genuine counting/deduplication rules to `event_serialization` or calculation notes |

## GitHub-content and site publication

The canonical machine contract remains versioned with the application and CGPT workflow. A generator must publish this human reference to `openKPIs-content/docs/catalog-field-reference.md` on contract changes. CI fails when the generated document differs from the canonical contract.

The site should expose a public `/field-guide` page generated from the same contract, linked from Contribute/Edit help and the footer. It must support filters for KPI, Metric, Dimension, and Event and distinguish Add fields, Edit-only fields, public fields, GitHub fields, and internal workflow fields. The page is documentation; it does not expose private user IDs or emails.

