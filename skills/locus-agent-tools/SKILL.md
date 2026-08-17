---
name: locus-agent-tools
preamble-tier: 1
version: 1.18.0
description: Use when an agent needs to connect to Locus over MCP, A2A, or REST for property and local-government context.
triggers:
  - locus agent tools
  - mcp or rest for locus
  - a2a locus
allowed-tools: Bash Read AskUserQuestion
license: MIT
---

# Locus Agent Capabilities

Use this skill to connect an agent to Locus when a place-based workflow needs local-government context: taxes, parcels, zoning, flood, environmental records, development activity, transportation projects, local policy, source coverage, or recurring monitoring. The fuller client guide is [`docs/AGENT_CAPABILITIES.md`](https://github.com/velinussage/locus/blob/main/docs/AGENT_CAPABILITIES.md).

Locus returns awareness and verification steps, not a verdict. Do not score, rank, predict, screen, value, or label a person, property, block, or neighborhood as safe/unsafe.

## What to remember first

- **51 national free tools are available with no payment or local coverage check; all 51 work directly from a geocodable US address (`locus_pfas_occurrence` resolves an address to serving system(s) via EPA's community water system service-area boundary layer, with per-boundary provenance/confidence, and still accepts explicit PWSIDs from `locus_water_systems`).** The live free catalog currently exposes 83 tools total. Use national lanes for rural addresses too, including flood, storm, wildfire, soil, groundwater-monitoring wells, cleanup, toxic-release, underground storage tank / leaking-tank, drought, water, PFAS occurrence and non-UCMR multimedia records, sewer-overflow/CSO context, NEI facility emissions, broadband, wetland, terrain, air-quality, governing-district, housing/economic, nearby-place, geotagged Wikipedia place/history context, safely licensed nearby Wikimedia Commons area media metadata, public-utility, county Medicare-spending, electricity/transmission, and aggregate traffic-crash context. Mirror-backed EPA lanes return explicit `not_ingested`, `partial`, or unavailable component states instead of treating missing data as a favorable result. Traffic crashes use a national FARS fatal-only fallback, with richer registry-derived coverage in Chicago, Virginia, and North Carolina; source classes and partial subsets stay explicit, counts below 3 are suppressed, and proximity never establishes route exposure or future probability.
- **National free tools cover all 50 states for geocodable US addresses.** Local lanes are wired jurisdiction by jurisdiction and are growing. Always expect national context. Treat local parcel, zoning, permit, tax, and development-case depth as coverage-dependent.
- **Start with `locus_place_facts` when lane availability says it is available.** It is the one-call address bundle for supported parcel areas: parcel facts, FEMA flood zone, governing districts, transportation context, and tax context where wired.
- **Use `locus_lane_availability` before paid calls.** It maps national, local, varies, not-covered, and degraded lanes, then gives per-paid-tool buy recommendations.
- **Treat partial trend coverage as a check-first signal.** `supported_partial` trend places appear in `lanes.varies` with low paid substance; buy `locus-local-trend-brief` only when `buyRecommendations[].substanceHere` is `medium` or better. Thin exact-radius results can return a `charged:false` data-sufficiency diagnostic instead of a paid brief.
- **The live paid catalog has 73 endpoints: 23 native paid tools plus 50 promoted dual-rail routes over focused free tools.** Most native single-call tools list at $0.05 to $0.10 USDC; surrounding-area analysis is $0.10, its report upgrade $0.15, local fiscal context is $0.05, `locus-property-flyer` is $0.49, and `locus-place-report-batch` is $0.25 per 3-50 address async job. Prefer free `/tools/call` for dual-rail lookups; prefer one composite over many micros (see product ladder). Call the live paid catalog or `locus_lane_availability` for current `priceUsdc`, then read the x402 challenge for exact price, chain, recipient, and schema before payment.

## Quick connect

Install this skill:

```bash
npx skills add velinussage/locus-agent-skill --skill locus-agent-tools
npx @velinussage/locus-agent-skill add
gh skill install velinussage/locus-agent-skill locus-agent-tools
```

Cold-agent maps (read these before copying tool lists):

- [API llms.txt](https://api.locus.report/llms.txt)
- [RFC 9727 API catalog](https://api.locus.report/.well-known/api-catalog)
- [Free catalog](https://api.locus.report/tools/list)

Remote MCP server:

```json
{
  "mcpServers": {
    "locus": {
      "type": "http",
      "url": "https://mcp.locus.report/mcp"
    }
  }
}
```

A2A and REST discovery:

- [Top agent endpoints](https://api.locus.report/.well-known/locus-agent-endpoints.json) - bundle-first menu for Before You Sign, Investor Diligence, and Policy + Environmental Brief.
- [Unified tool catalog](https://api.locus.report/.well-known/locus-tools.json) - every free and paid schema, price, route, and free/paid counterpart.
- [Agent Card](https://api.locus.report/.well-known/agent-card.json) - A2A skills and message endpoint.
- [Free tool catalog](https://api.locus.report/tools/list) - free tool schemas, descriptions, and read-only metadata.
- [llms.txt](https://api.locus.report/llms.txt) - first-hop agent map for the free catalog and skill.
- [RFC 9727 API catalog](https://api.locus.report/.well-known/api-catalog) - standard service-desc links.
- [Detailed paid tool index](https://api.locus.report/.well-known/ai-tool/index.json) - current prices, schemas, and manifests.
- [Well-known skill](https://api.locus.report/.well-known/skill.md) - this operating guide from the API origin.
- [MCP catalog](https://mcp.locus.report/catalog) - searchable tool catalog.
- [API base](https://api.locus.report) - health and discovery links.

Buyers do not need a Locus or Coinbase Developer Platform account API key. Free tools are open. Paid tools return a live x402 or advertised MPP challenge. `PAYMENT-SIGNATURE` is a signed payment credential, not an account API key.

For broad requests, start with the three bundles in the top-agent manifest. For exact lanes, use the full catalog and keep the $0.01 promoted atomic routes available. Promoted paid atomics use REST; their free underscore counterparts are callable through `POST /tools/call` and MCP. Free executor names use underscores (`locus_zoning`); paid REST slugs use hyphens (`locus-zoning`). Use `catalogId` as the unique catalog key. Use `canonicalId` only for normalized search and grouping. Follow `counterparts[]` to move between related free and paid routes, and execute the entry's exact `callName`.

The live catalogs are authoritative for tool names, schemas, prices, and endpoints. Do not copy stale tool definitions into prompts.

## Surrounding-area orchestration route

When the buyer wants more than the free `locus_surrounding_parcels` primitive and needs one composed view of surrounding parcels, zoning, development cases, permits, legislation, transportation/capital projects, environmental mechanisms, and optional dated aerial evidence, load the separate `locus-surrounding-area-analysis` skill. It owns the paid two-stage workflow and its recovery states. Keep this general capability skill on free discovery and the broader public tool surface; do not copy the paid workflow into ordinary Locus calls.

## Workflow and coverage tools, know the difference

- **`locus_suggest_workflow { "place": "...", "intent": "homebuyer_due_diligence" }`** is the free deterministic planner. It returns a ranked endpoint plan with reasons, estimated costs, free/paid rail, national/local scope, and place availability. Use it first when the intent is broad or the right endpoint is unclear. Accepted intents are `renter_due_diligence`, `homebuyer_due_diligence`, `small_business_siting`, `community_organizer_journalist`, `commercial_tenant_due_diligence`, `land_investor_due_diligence`, `developer_predevelopment`, `environmental_research`, `short_term_rental_operations`, and `hoa_board_governance`; short natural-language equivalents are also normalized. These extra planner lanes route tools only; they do not widen the four safety-versioned paid brief claim intents.
- **`locus_coverage_check { "place": "..." }`** asks whether Locus has source coverage for a jurisdiction at all. Use it for a broad city, county, ZIP, or address scope check.
- **`locus_lane_availability { "place": "..." }`** is the per-address capability map. It returns which exact tools are national, local, varies, not covered, or degraded, plus buy signals for paid tools. Call this before address-specific local lanes and before paying.
- **`locus_coverage_map {}`** returns the whole registry view. Use it when an agent needs breadth, not one address.

## Storefront and commercial-tenant context

- Keep listing discovery outside Locus and preserve listing claims as unverified leads in source order. Do not rank sites.
- Use free `locus_workplace_employment_context` for annual Census workplace jobs and broad industries near a selected address when the active state snapshot is available. Jobs are not shoppers, foot traffic, visits, sales, customer demand, or a forecast.
- The tool can return `not_ingested`, `stale_snapshot`, `source_unavailable`, `partial`, or a bounded no-match. Do not turn any of these into zero employment.
- For a user-selected premises, call paid `locus-transaction-follow-up` with `intent: "commercial_tenant_due_diligence"`. Add `requestedAreaContext: ["workplace_employment"]` only when the user asks for that context.
- `area_operating_context` is supplemental. It cannot make a thin exact-premises packet chargeable.
- Use `locus_wikipedia_place_context` for nearby geotagged English Wikipedia articles when secondary place or local-history context would help. It returns bounded plain-text extracts plus page/revision provenance. Wikipedia is community-edited and never substitutes for an official record, current-condition evidence, or tenant history.
- Use `locus_wikimedia_commons_area_context` for safely licensed geotagged media metadata when broader nearby area context would help. It deliberately does not require a place-name match, so every item remains `nearby_context`. Proximity does not prove that media depicts the premises or represents the area. A no-match describes only the bounded query. Never use counts, exclusions, distance, freshness, or absence to score, rank, screen, or infer area quality, desirability, condition, or demand. The response does not grant permission to store, transform, or redistribute pixels. Verify each source page and comply with its item-level license.

## Exact-place and paid-call guardrails

- **Use the exact user string first.** Do not append a city, county, ZIP, or better-covered market unless the user provided it or confirms it.
- **Never substitute a richer-coverage jurisdiction.** If `17 E Camden` resolves to Chatham County but `17 E Camden St, Raleigh, NC` has richer civic lanes, the answer stays Chatham/parcel-only until the user confirms Raleigh.
- **Trust exact parcel/place resolution over coverage richness.** Coverage tells you which lanes are available for a resolved place; it does not license geocoding toward another jurisdiction.
- **Ask before paid calls when jurisdiction is ambiguous.** If candidate variants resolve to different counties/cities, stop and ask the user to confirm the intended jurisdiction.
- **First line of every report:** `Resolved as: <displayName> (<jurisdictionId>) via <resolution>; parcel status: <parcelStatus>; civic lane status: <civicLaneStatus/coverageStatus>.`
- **Parcel-only mode:** if civic lanes are unsupported but parcel facts are verified/available, use parcel, zoning-if-available, tax-rate/district, parcel-transfer, tax-distress, nearby-places, national hazard/environmental, and verify-next tools. Do not lead with service requests or generic place-report counts, and do not treat missing civic lanes as "no activity."

## Workflow

1. **Plan broad intents.** Call `locus_suggest_workflow` with the exact place and the closest supported intent when the right endpoint is unclear.
2. **Discover exact schemas.** Use the table below for common intents. The live catalog, `locus_search_tools` over MCP or `GET /tools/list` over REST, is authoritative for the full current set and exact schemas.
3. **Resolve the exact place first.** Call `locus_coverage_check` and `locus_lane_availability` with the exact user string. Compare the resolved jurisdiction to any user-supplied city/county/state.
4. **For a broad address question, call `locus_place_facts` first if available.** It often replaces several separate calls. If lane availability marks it not covered, fall back to national free tools or parcel-only mode.
5. **For local depth, check availability for the exact place.** Call only the lanes it marks `available`. For `varies`, call the free tool once and let its returned coverage/status decide.
6. **Run the smallest tool by intent.** Use `locus_execute` over MCP, `/a2a/v1/message:send` with a DataPart over A2A, or `POST /tools/call` over free REST. Prefer one targeted free tool over a paid bundle when the question is narrow.
7. **Ground every fact.** Answer only from returned artifacts. Include source names, links or locators, fetched timestamps where present, and caveats.
8. **Pay only on explicit authorization.** A paid tool returns an x402 challenge. Show price, chain, recipient, and tool, then retry only after the user approves.
9. **Follow property-update diagnostics exactly.** On `409 clarification_required`, inspect the response before retrying. If `retryInput` is present, ask the user to confirm the matched subject and then resend that object as the next request. If `retryInput` is absent, ask the user for a corrected exact address and construct a new request from it. Never resend the original ambiguous address. On `insufficient_current_context`, use the returned wider radius or choose one of `alternativeTools`; those alternatives are separate paid calls and still require their own preflight and authorization.
10. **Chain PDF to flyer before video completion.** Poll the property-update job. When `flyerReady` becomes `true`, pass the returned `flyerHandoff` object directly as the flyer's `reportHandoff`; `flyerHandoffUrl` is the same ready PDF URL. Do not build a `share_` proof yourself or prefix the private job token. The server derives a separate report-scoped share capability.

## Reading `locus_lane_availability`

`locus_lane_availability` returns a top-level result with `place`, `resolved`, `jurisdiction`, `lanes`, `buyRecommendations`, `recommendedCallOrder`, `relatedTools`, and `warnings`.

Statuses and buckets:

- **`lanes.national[]`** - free national or metadata tools. These are usable for any resolved US address.
- **`lanes.local[]`** - tools with wired sources here, including paid national bundles when applicable.
- **`lanes.varies[]`** - source may resolve. Call the free tool to confirm before relying on it or paying.
- **`lanes.notCovered[]`** - skip it. Tell the user this lane is not wired and offer `locus_request_coverage`.
- **`lanes.degraded[]`** - upstream source is temporarily failing or reduced. Use the suggested fallback.
- **`buyRecommendations[]`** - paid tool guidance with `priceUsdc`, `substanceHere`, `rationale`, endpoint, and manifest.

Trimmed response example for a rural Montana ZIP:

```json
{
  "ok": true,
  "tool": "locus_lane_availability",
  "result": {
    "place": "59047",
    "resolved": true,
    "jurisdiction": {
      "jurisdictionId": "us-mt-park",
      "displayName": "Park County, MT",
      "stack": { "state": "MT", "county": "Park County" }
    },
    "lanes": {
      "national": [
        { "tool": "locus_flood_zone", "what": "FEMA flood-zone designation at the point", "access": "free" },
        { "tool": "locus_radon_zone", "what": "EPA radon zone for the county", "access": "free" },
        { "tool": "locus_wildfire_risk", "what": "FEMA NRI wildfire risk rating", "access": "free" },
        { "tool": "locus_representatives", "what": "Cited state + federal officials for the point", "access": "free" }
      ],
      "varies": [
        { "tool": "locus_zoning", "access": "free", "why": "point zoning may resolve; rich coverage only in wired counties" }
      ],
      "notCovered": [
        { "tool": "locus_place_facts", "access": "free", "why": "needs a wired parcel backbone", "requestTool": "locus_request_coverage" }
      ],
      "degraded": []
    },
    "buyRecommendations": [
      { "slug": "locus-place-report", "priceUsdc": "0.05", "substanceHere": "low", "rationale": "Coverage varies. Confirm the free component lanes first." },
      { "slug": "locus-environmental-context", "priceUsdc": "0.05", "substanceHere": "medium", "rationale": "Wired national EPA/SDWIS sources resolve here." }
    ],
    "warnings": [
      "Not covered does not mean no records exist. It only means Locus has no wired source yet."
    ]
  }
}
```

A paid tool flagged not covered returns a free diagnostic, never a payment challenge.

## Free tools by question, arguments, and output

Use the exact JSON shapes below as safe defaults. If a tool also accepts `latitude` and `longitude`, use them together to skip geocoding. The live `GET /tools/list` schema wins if it differs.

### Start here and coverage

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| Which endpoints fit this place and intent? | `locus_suggest_workflow` | `{ "place": "600 E 4th St, Charlotte, NC", "intent": "homebuyer_due_diligence" }` | Ranked free/paid endpoint plan, estimated costs, scope, reasons, and place-availability hints. Planning only; confirm with coverage tools. |
| Which tools will return data here? | `locus_lane_availability` | `{ "place": "600 E 4th St, Charlotte, NC" }` | Jurisdiction, `lanes.national/local/varies/notCovered/degraded`, paid buy signals, warnings. |
| Is this city/county/ZIP in source coverage? | `locus_coverage_check` | `{ "place": "Raleigh, NC" }` | Resolved jurisdiction, supported/partial/discovery status, verified sources, missing source gaps. |
| What is the whole coverage registry? | `locus_coverage_map` | `{}` | Registry-level coverage inventory for tools and jurisdictions. |
| Request coverage for a missing place | `locus_request_coverage` | `{ "place": "Park County, MT" }` | Acknowledgement and demand signal. No records. |
| Inspect official source cards | `locus_source_card_check` | `{ "place": "Raleigh, NC" }` or `{ "jurisdiction": "us-nc-raleigh" }` or `{ "cardId": "us-nc-durham:permits" }` | Source-card status, provenance, endpoint, verification method, timestamp. |
| Verify a citation URL | `locus_verify_citation` | `{ "sourceUrl": "https://...", "recordId": "optional", "jurisdiction": "optional" }` | Whether a citation matches a known source card, with provenance context. |
| What policy sources govern here? | `locus_policy_sources` | `{ "place": "Raleigh, NC" }` | State, county, city policy-source list, legal geographies, source links. |
| Read aggregate coverage demand | `locus_coverage_demand` | `{}` | Aggregate requested-coverage demand, not place records. |

### Paid product ladder (use one path)

| Question | Prefer | Do not open with |
|---|---|---|
| Free snapshot | `locus_place_facts` (free) | Paid dual of the same |
| Pre-sign parcel + trend + policy | `locus-before-you-sign` ($0.07) | Three separate briefs |
| Neighbors, topology, multi-lane surrounding evidence | `locus-surrounding-area-analysis` ($0.10) ± report ($0.15) | `locus-ownership-loop` alone or many micro duals |
| EPA proximity bundle | `locus-environmental-context` ($0.05) | Parallel toxic/RCRA/water duals |
| Compiled place artifact | `locus-place-report` ($0.05) | Stitching free tools into a fake report |
| Recent official change + media | `locus-property-update` ($0.10) | Flyer first |

Full inventory decisions: `docs/PAID_TOOL_INVENTORY.md`. Paid index entries also carry `seeAlso` for overlap routing.

### Paid tools by endpoint

Use these only after `locus_lane_availability` or the paid index says the call has substance for the exact place. The live paid index is authoritative for current prices and schemas.

| Endpoint | Price | Use when | Free diagnostic behavior |
|---|---:|---|---|
| `POST /api/locus-surrounding-area-analysis` | `$0.10` | Buyer needs one stored multi-lane surrounding packet: topology-aware parcels, zoning, development, permits, legislation, capital/transport, environmental baseline, optional aerial. | Unstable subject, missing surrounding-parcel foundation, or under two completed components returns `charged:false`. |
| `POST /api/locus-surrounding-area-report` | `$0.15` | HTML+PDF upgrade of an active surrounding-area packet within the 24h upgrade window. | Invalid/expired proof, second report, or render failure does not charge. |
| `POST /api/locus-place-report` | `$0.05` | Agent needs one compiled cited property-context artifact for an address or ZIP. The artifact confirms the matched subject, lists every source, and carries an honest coverage ledger of which lanes were checked (covered, empty, or not wired) plus a machine-readable `report.index`/`access` envelope. | Unsupported or discovery-only places return no-charge diagnostics. |
| `POST /api/locus-property-update` | `$0.10` | Agent needs an async exact-address decision check of recent or scheduled official-record changes, nearby activity, and physical comparability, with a shareable report, PDF, and temporary video. | Ambiguous, thin, or unsupported inputs return `charged:false`; on `clarification_required`, confirm and resend `retryInput`. Poll the job and, once `flyerReady:true`, use `flyerHandoff` immediately without waiting for video. |
| `POST /api/locus-property-flyer` | `$0.49` | After obtaining a proof-gated PDF URL + expiry from a report workflow, the agent wants a general 4:5 property shareable whose top-right QR opens that PDF. Pass the property-update job's copy-ready `flyerHandoff` as the flyer-specific `reportHandoff`; do not synthesize a proof or reuse the video render contract. A licensed subject image is optional (Locus can generate an aerial hero and parcel outline). | Missing runtime, usable research, imagery, or premium model output returns `charged:false`; expired handoffs and prohibited claims fail before model spend. A `property_flyer_claim_rejected` error identifies `rejectedField`, optional `rejectedFeatureIndex`, and the stable `reason`. |
| `POST /api/locus-place-report-batch` | `$0.25` | Agent has a 3-50 address portfolio and wants one async job plus one settlement. | If all items are unsupported or discovery-only, no charge. Unsupported items inside a paid job remain item-level diagnostics. |
| `POST /api/locus-local-trend-brief` | `$0.05` | Agent needs permit, 311, or code-case local-change series where the registry has enough source coverage. | Unsupported, discovery-only, or insufficient-data places return `charged:false` diagnostics. |
| `POST /api/locus-local-policy-brief` | `$0.07` | Agent needs property-relevant bills, agendas, ordinances, tax, fee, bond, housing, or permit-change policy context. | Unsupported places return no-charge diagnostics. |
| `POST /api/locus-local-fiscal-context` | `$0.05` | Agent needs separate cited local-government fiscal/audit observations and a bounded five-year trend from an offline verified snapshot. Peer comparison remains disabled until Locus can verify complete official cohorts. | Missing, stale, or thin snapshots return `charged:false`; never returns an integrity, corruption, government-quality, credit, or place score. |
| `POST /api/locus-before-you-sign` | `$0.07` | Agent needs a pre-decision bundle over parcel, trend, and policy components for one street address. | Weak component readiness returns no-charge component diagnostics. |
| `POST /api/locus-environmental-context` | `$0.05` | Agent needs address-level EPA TRI/RCRA/SDWIS/radon public-record context ranked by distance where possible. | Unsupported or unresolvable inputs return no-charge diagnostics. |
| `POST /api/locus-air-quality-history` | `$0.05` | Agent needs nearby EPA AQS annual monitor summaries, NOAA HMS smoke-over-point days, and separately labeled CDC modeled PM2.5 gap-fill. | No substantive monitor/smoke/modeled rows or missing mirror coverage returns `charged:false`; never substitutes missing years with “good air.” |
| `POST /api/locus-property-tax` | `$0.05` | Agent needs a residential US property-tax artifact with assessed value, annual tax, tax history, effective rate, and provenance. | Commercial, uncovered, or unresolvable addresses return `charged:false` diagnostics pointing to the free `.gov` tax lanes or place report. |
| `POST /api/locus-rent-estimate` | `$0.05` | Agent needs a third-party residential long-term rent estimate, range, comparable count, and HUD FMR area anchor. | No estimate, no comparables, commercial use, missing key, or uncovered inputs return `charged:false`. Not a Locus-authored valuation. |
| `POST /api/locus-valuation-challenge` | `$0.10` | Agent wants to challenge a caller-supplied property price against cited sale, parcel, permit, hazard, tax, zoning, and policy evidence without Locus creating a price. | Fewer than two substantive cited sections return `charged:false`. |
| `POST /api/locus-road-access` | `$0.05` | Agent needs nearest mapped public-road proximity from an address/point as an early access screen. | Unresolved address or total source failure returns `charged:false`. Never a legal-access, easement, frontage, or landlocked determination. |
| `POST /api/locus-power-water-evidence-pack` | `$0.05` | Agent needs pre-development proximity/context from HIFLD/EIA power, EPA public-water-system, and FCC broadband sources. | Missing substantive evidence suppresses charge. Never claims capacity, interconnection, service availability, timing, or cost. |
| `POST /api/locus-landslide-diligence` | `$0.05` | Agent needs separate USGS documented inventory history and exact source-native n10 model-cell evidence. | Unless both components answer, returns `charged:false`. Never a probability, parcel stability finding, engineering assessment, or safety label. |
| `POST /api/locus-permit-closeout-check` | `$0.05` | Agent needs exact-parcel Raleigh/Durham permit status plus source-published closeout/occupancy-document evidence. | Uncovered, uncertain, unavailable, unpublished, and no-exact-match states are charge-suppressed. Not condition, compliance, suite/use permission, or closing approval. |
| `POST /api/locus-transaction-follow-up` | `$0.10` | Agent needs one explicit homebuyer, land-investor, developer-predevelopment, or commercial-tenant follow-up packet with cited handoffs. Commercial tenants may request `requestedAreaContext: ["workplace_employment"]`. | Profile-specific component/group thresholds control chargeability. Supplemental `area_operating_context` never counts toward the premises threshold. Caller prose cannot select intent or evidence tools; no readiness, buildability, occupancy, value, or transaction verdict. |

### Best first call for supported address context

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| Give me one broad free snapshot for this address | `locus_place_facts` | `{ "address": "1 E Edenton St, Raleigh, NC 27601" }` | One-call bundle: parcel facts where wired, FEMA flood zone, governing districts, transportation context, property-tax context where wired, citations, and caveats. Start here when available. |
| Parcel facts only | `locus_parcel_lookup` | `{ "address": "123 Oak Park Dr, Cary, NC 27519" }` | PII-safe parcel or place facts such as land use, acreage, year built where present, derived county/municipality, provenance. |
| Multiple caller-supplied parcels | `locus_parcel_set` | `{ "addresses": ["addr 1", "addr 2"] }` | Normalized facts for up to 25 supplied addresses in an assemblage. |
| Which public-record checks should I run before site diligence? | `locus_predevelopment_feasibility` | `{ "address": "...", "projectType": "multifamily" }` | Ordered Locus-tool and official-source plan for parcel/zoning, entitlements, utilities, access, environmental constraints, policy, fees, and permits. Prompt-mode; does not decide feasibility. |
| Find candidate-site research paths for an area/spec | `locus_development_site_discovery` | `{ "area": "Wake County, NC", "projectType": "industrial" }` | Open-ended candidate-parcel discovery source map. Does not execute searches, select a parcel, or rank sites. |
| Compare known properties without ranking them | `locus_compare_properties` | `{ "properties": [{ "address": "..." }, { "address": "..." }] }` | Side-by-side public-record comparison plan across parcel, zoning, permits, utilities, access, hazards, tax/fees, and market activity. |

### National free content tools

These work for geocodable US addresses at no cost. Some accept address directly. Others need county, state, ZIP, or FIPS. Use `locus_lane_availability` or any geocoder/FIPS resolver to derive those arguments when needed.

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| FEMA flood zone at a point | `locus_flood_zone` | `{ "address": "600 E 4th St, Charlotte, NC" }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Flood zone code, SFHA flag, panel or DFIRM identifiers when returned, plain-language zone context, provenance, verify-next steps. |
| Nearby flood gauges | `locus_flood_gauges` | `{ "address": "...", "radiusMeters": 20000 }` | Nearby USGS/NWS gauges, latest stage/flow observations when available, threshold metadata, source links. |
| Governing districts | `locus_governing_districts` | `{ "address": "..." }` | Congressional, state senate, state house, county, municipality, and other governing geographies when resolved. |
| Representatives | `locus_representatives` | `{ "address": "..." }` | Cited federal and state officials for the address, districts, party/office/contact links where available. |
| Wildfire risk | `locus_wildfire_risk` | `{ "address": "..." }` | FEMA NRI tract-level wildfire rating, score, expected annual loss fields where returned, provenance, caveats. |
| Current air quality and smoke | `locus_air_quality_current` | `{ "address": "...", "radiusKm": 80 }` or `{ "latitude": 35.78, "longitude": -78.64 }` | Nearest preliminary EPA AirNow monitor AQI readings plus NOAA HMS smoke polygons intersecting the point. Separate from wildfire hazard/perimeters; no causal attribution or safe/unsafe score. |
| Environmental history bundle | `locus_environmental_history` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` | EPA TRI, RCRA, SDWIS water systems, radon zone, and event-history verify links. Read `radonHeadline` and `addressProximity` first; `countyBackground` and legacy county fields are broad context only. |
| Cleanup sites | `locus_cleanup_sites` | `{ "address": "...", "radiusMeters": 5000 }` | EPA Superfund, NPL, brownfield, or cleanup-site records nearby with program/status fields and official profile links. |
| Underground storage tanks and leaking-tank releases | `locus_underground_storage_tanks` | `{ "address": "...", "radiusMeters": 1500 }` | Nearby EPA UST Finder facility records and reported leaking-tank releases, kept separate with per-component availability. A registered tank is not a leak, and proximity does not establish parcel contamination, exposure, or liability; an unavailable component is not a verified zero. |
| Toxic releases | `locus_toxic_releases` | `{ "state": "MT", "county": "Park County" }` | EPA TRI facilities for the county, chemical-release reporting fields where available, profile links. |
| RCRA hazardous-waste handlers | `locus_rcra_handlers` | `{ "state": "MT", "countyFips3": "067" }` or `{ "state": "MT", "zip": "59047" }` | RCRAInfo handlers, generator status/activity fields, coordinates, ECHO verify links. |
| EPA radon zone | `locus_radon_zone` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` | EPA county radon zone 1/2/3, county basis, testing/disclosure caveat, provenance. |
| Public water systems | `locus_water_systems` | `{ "state": "MT", "county": "Park County" }` | Active public water systems serving the county, PWSIDs, SDWIS/ECHO links. County membership does not prove which system serves an address. |
| Groundwater monitoring-well context | `locus_groundwater_context` | `{ "address": "...", "radiusMeters": 5000 }` | Nearby USGS monitoring locations, aquifer codes/names, well-construction depth distributions, and static water-level observations, returned in distance bands without location names or exact coordinates. Not a domestic-well depth, yield, potability, contamination, or cost prediction. |
| EPA Sole Source Aquifer designation | `locus_sole_source_aquifer` | `{ "address": "..." }` | EPA mapped designation containment, designation names, and Federal Register citations. A designation means EPA reviews federally funded projects in the area under SDWA 1424(e); it is not a contamination or water-quality finding and does not itself restrict a private owner. |
| PFAS occurrence monitoring | `locus_pfas_occurrence` | `{ "address": "..." }` or `{ "pwsids": ["NC0319015"] }` | EPA UCMR 5 PFAS detection/non-detect summaries, highest measured occurrence, sample dates, vintage, and explicit out-of-coverage PWSIDs. An address is matched point-in-polygon against EPA's community water system service-area boundaries; each matched system carries boundary provenance and confidence (provider-supplied vs machine-modeled, verification status) - containment is strong evidence of the serving utility, not proof, and a point in no polygon returns an explicit no_service_area result. System entry-point monitoring only; not a tap sample, MCL compliance finding, exposure determination, or clean-water conclusion. Lithium is excluded. |
| Non-UCMR PFAS multimedia records | `locus_pfas_multimedia_nearby` | `{ "address": "...", "radiusMeters": 10000 }` | EPA PFAS Analytic Tools environmental-media, wastewater, TRI, Superfund/federal-site, and industry records from reviewed mirror exports. Measured/reported records stay separate from potential-handling flags; proximity does not establish source, migration, exposure, or parcel contamination. |
| Combined-sewer outfalls and reported events | `locus_sewer_overflow_nearby` | `{ "address": "...", "radiusMeters": 5000, "lookbackDays": 365 }` | EPA's CSO outfall inventory plus separately labeled sewer-overflow/bypass event reports. An outfall is not an event, and an event-stream miss is not proof of no overflow. |
| NEI facility emissions | `locus_nei_emissions_nearby` | `{ "address": "...", "radiusMeters": 5000, "reportingYear": 2023 }` | EPA National Emissions Inventory pollutant rows queried from the official facility-summary coordinates, with EIS/program ids preserved and no inferred FRS crosswalk. Periodic inventory estimates only, not monitor readings, ambient concentrations, exposure, or a score. |
| Drinking-water violations | `locus_drinking_water_violations` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` or `{ "pwsids": ["MT0000000"] }` | SDWIS/ECHO violation and enforcement rows, system ids, caveats that this is not proof of tap-water quality. |
| NOAA storm events | `locus_storm_events` | `{ "stateFips": "30", "countyName": "Park", "countyFips3": "067" }` | NOAA Storm Events history, event type, dates, damage estimates, narratives, and source links. Use `stateFips`, not `state`. |
| FEMA disaster and NFIP history | `locus_fema_events` | `{ "address": "..." }` or `{ "state": "MT", "countyFips3": "067", "zip": "59047" }` | FEMA disaster declarations by county, NFIP claims by ZIP, preliminary FIRM panel context where available. |
| Current drought status and recent history | `locus_drought_status` | `{ "address": "...", "weeks": 8 }` | Weekly US Drought Monitor county percentages by D0-D4 category plus recent weeks. County-level drought mapping does not establish parcel water supply, well yield, current utility restrictions, or a forecast. |
| HUD fair-market rents | `locus_fair_market_rents` | `{ "stateFips": "30", "countyFips3": "067", "year": 2026 }` | HUD FMR values by bedroom count and county/FIPS basis. |
| Qualified Opportunity Zone | `locus_opportunity_zone` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Whether the point is in a Treasury/IRS Qualified Opportunity Zone tract, cited to HUD. A `not_designated` zero-hit is a valid designation answer, not a coverage failure. Not tax or investment advice. |
| Seismic design parameters | `locus_seismic_design` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | USGS NEHRP/ASCE 7 seismic design parameters for a point. Hazard data only, not a safety verdict. |
| County unemployment trend | `locus_unemployment` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Recent BLS LAUS county unemployment trend. Reported statistic only, not an area-quality label. |
| State house-price index | `locus_house_price_index` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Latest state-level FHFA All-Transactions House Price Index and year-over-year change via FRED. Not an appraisal or value estimate. |
| Broadband availability map | `locus_broadband_check` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Official FCC National Broadband Map link and current data vintage. Provider-reported availability; no fast/slow/good/bad verdict. |
| Mapped wetland overlap/proximity | `locus_wetland_context` | `{ "address": "...", "radiusMeters": 1500 }` | FWS NWI mapped-wetland overlap and nearby polygon evidence. Never a delineation, jurisdictional determination, parcel boundary, or permit decision. |
| Elevation and sampled terrain | `locus_terrain_profile` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | USGS EPQS/3DEP elevation plus sampled cardinal terrain profile/slopes. Never a survey, drainage, grading, or engineering conclusion. |
| Transportation-noise proximity facts | `locus_noise_proximity` | `{ "address": "..." }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Straight-line proximity to public-use airports and mapped major-road/rail centerlines. No decibel estimate or quiet/loud/safe/unsafe label. |
| Nearby places and amenities | `locus_nearby_places` | `{ "address": "...", "radiusMeters": 800 }` | OpenStreetMap nearby amenities/places with categories, distance, and OSM provenance. |
| Nearby Wikipedia place/history context | `locus_wikipedia_place_context` | `{ "address": "...", "radiusMeters": 1000, "limit": 10 }` | Geotagged English Wikipedia articles with bounded plain-text extracts and page/revision provenance. Secondary community-edited context only; not an official record, subject-property match, or current-condition finding. |
| Nearby Wikimedia Commons area media | `locus_wikimedia_commons_area_context` | `{ "address": "...", "radiusMeters": 2000, "limit": 15 }` | Safely licensed geotagged image metadata and source links labeled `nearby_context`. No name match is required. Proximity does not prove that media depicts the premises or is representative. Locus stores no pixels, and the response grants no storage, transformation, or redistribution permission. |
| Nearest emergency services and utilities | `locus_public_utilities` | `{ "address": "...", "radiusMeters": 5000 }` | Nearest mapped OpenStreetMap fire station, hospital/clinic, police, fire hydrant, electric substation, and water tower with straight-line distances and counts. Hydrants/infrastructure are often unnamed and still reported. Mapped facility distances only, never a fire-protection rating, insurance determination, or safety verdict. |
| Transmission lines + state electricity-price context | `locus_electricity_context` | `{ "address": "...", "radiusMeters": 10000 }` | Up to 10 nearby mapped HIFLD transmission lines plus latest/year-ago EIA statewide residential average price when `EIA_API_KEY` is configured. No owner/substation fields. Price is independent of line proximity and is never a property tariff, bill, service, capacity, reliability, or interconnection claim. |
| County Medicare fee-for-service spending | `locus_medicare_spending` | `{ "address": "..." }` | Latest/prior CMS county aggregate: Original Medicare fee-for-service beneficiary count, actual and standardized per-capita payment, and standardized year-over-year change. Not a provider price, individual bill, premium, care-quality/access measure, health inference, score, or property signal. |
| Aggregate traffic-crash context | `locus_traffic_crash_context` | `{ "address": "...", "radiiMeters": [200, 500, 1000], "lookbackYears": 3 }` | Small-cell-suppressed police-reported crash counts by official severity and mode. Registry coverage: Chicago all-reported, Virginia reportable-threshold, North Carolina nonmotorist and K+A subsets kept separate, plus nationwide FARS fatal-only fallback. No raw rows, examples, exact points, party details, score, forecast, or place verdict. |
| Landslide history + susceptibility context | `locus_landslide_context` | `{ "address": "...", "searchRadiusMeters": 1000 }` or `{ "latitude": 35.78, "longitude": -78.64 }` | USGS v3 bounded inventory point/polygon records plus a separately labeled exact source-native n10 90-meter cell where the active snapshot publishes one. Inventory no-match, model out-of-footprint, stale/unavailable snapshot, and source failure remain distinct. The 0–81 model value is not probability or parcel engineering. |
| FEMA NFIP flood-insurance claims (county) | `locus_nfip_claims` | `{ "address": "..." }` or `{ "latitude": 25.76, "longitude": -80.19 }` | FEMA OpenFEMA NFIP redacted flood-insurance claims aggregated to the county: total claim count, total paid, year range, and top rated flood zones. Redacted (no address/PII), county-level historical fact, never a prediction or risk score. |
| Historical wildfire perimeters | `locus_wildfire_history` | `{ "address": "..." }` or `{ "latitude": 38.6, "longitude": -122.5 }` | WFIGS interagency historical wildfire perimeters within about 5 miles: incident name, category, acres, discovery year. Realized-event records (complements the modeled `locus_wildfire_risk`); most non-Western points return none, an honest `out_of_coverage`. |
| Soil context and site constraints | `locus_soil_context` | `{ "address": "..." }` or `{ "latitude": 35.9, "longitude": -78.9 }` | USDA NRCS SSURGO soil suitability at a point: NRCS engineering ratings (Very/Somewhat/Not limited, with limiting features) for dwellings, septic absorption fields, and shallow excavations, plus drainage, hydric, slope, shrink-swell (linear extensibility), taxonomic order, and farmland class. Soil-map-unit-level screening context for septic/foundation/drainage/buildability, never a geotechnical, perc-test, permit, or safe/unsafe/buildable verdict. Developed/urban points often return `out_of_coverage` (no soil component). |
| Data-center or large-development watch prompts | `locus_data_center_watch` | `{ "address": "..." }` or `{ "state": "NC", "county": "Wake County", "municipality": "Raleigh" }` | Bounded official-source query pack and lead-discovery prompts for large projects, data centers, utility/water/planning sources. |
| Local ordinance and storefront-process leads | `locus_ordinance_leads` | `{ "address": "...", "topics": ["occupancy", "fee_schedule", "signage", "sidewalk_cafe", "published_review_timeline"] }` or `{ "place": "Raleigh, NC" }` | Jurisdiction-locked first-party query pack and reviewed process pointers. Numeric fees/timelines require an exact dated official source; published timelines are guidance, not predictions. Leads only, no restricted code body, not legal advice. |
| Coastal county catalog | `locus_coastal_county_catalog` | `{}` or `{ "county": "new-hanover-nc" }` | Catalog of supported coastal source packs, overlays, source leads, and limitations. |

Temporarily degraded national tools may appear in `lanes.degraded` when an upstream source is failing for every point. If `locus_regulated_facility_compliance` appears there during a future EPA ECHO outage, use `locus_environmental_history` and `locus_toxic_releases` as the functional fallback; if `locus_housing_stock` appears there during a future Census outage, use `locus_fair_market_rents` for housing-cost context meanwhile. Otherwise those tools are callable national lanes.

### Local or coverage-dependent free tools

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| Zoning district and overlays | `locus_zoning` | `{ "address": "..." }` or `{ "latitude": 35.78, "longitude": -78.64 }` | Governing zoning district, overlays/planning context where wired, source links. Rich coverage only in wired jurisdictions. |
| Development or rezoning nearby | `locus_development_cases` | `{ "address": "...", "radiusMeters": 1500 }` | Nearby development/rezoning case records, statuses, dates, identifiers, citations where wired. |
| Public capital projects nearby | `locus_capital_projects` | `{ "address": "...", "radiusMeters": 1609 }` | Government capital projects, public works, assessments where wired, with project ids and source links. |
| Building permits in supported metros | `locus_metro_permits` | `{ "city": "chicago", "latitude": 41.878, "longitude": -87.629, "radiusMeters": 1000, "sinceDate": "2025-01-01" }` | Nearby issued permit records, permit number/type/status, issue date, reported cost, source URL. Wired verified metros: Chicago, Austin, Seattle, San Francisco, Los Angeles, New York, Mesa, Miami (City of Miami only), Nashville, Washington DC, San Antonio, Phoenix, Denver, Fort Worth, Buffalo, and Detroit. Other metros return an out-of-coverage diagnostic, not a silent empty. |
| Transportation projects and traffic counts | `locus_transportation_context` | `{ "address": "...", "radiusMeters": 2000 }` | State DOT funded projects, traffic-count stations, routes, statuses where wired. |
| Public water/sewer service-area screen | `locus_utility_service_check` | `{ "address": "..." }` | Whether the point falls inside mapped public water and sewer service-area polygons where county sources are wired, with provider names when safely published. Polygon membership does not prove service availability, capacity, connection rights, timing, or cost for a parcel. |
| Transit stops and routes | `locus_transit_context` | `{ "address": "...", "radiusMeters": 400 }` | Transit stops, routes, shelter/ADA fields, headways where supported. Wired transit agencies span about 15 metros: New York (MTA), Los Angeles (LA Metro), Houston (METRO), Charlotte (CATS), Miami-Dade, Nashville (WeGo), Washington DC (WMATA), Dallas (DART), Philadelphia (SEPTA), Atlanta (MARTA), San Antonio (VIA), Seattle (King County Metro), Phoenix (Valley Metro), Denver (RTD), and Raleigh (GoRaleigh). Other areas return no wired transit lane. |
| Recent nearby parcel transfers | `locus_parcel_transfers` | `{ "address": "...", "radiusMeters": 1500, "monthsBack": 12 }` | Recorded sales/transfers near the point where parcel-sale sources are wired. |
| Property-tax rates | `locus_property_tax_rates` | `{ "place": "Wake County, NC" }`, `{ "place": "Nashville, TN" }`, `{ "place": "Austin, TX" }`, or `{ "place": "Tampa, FL" }` | Adopted rate components and jurisdiction basis from official tables where wired: NC statewide plus selected Nashville/Davidson TN, Austin/Travis TX, and Tampa/Hillsborough FL adapters. Other jurisdictions return an official-source prompt pack, not a server-emitted rate number. |
| Property-tax estimate | `locus_property_tax_estimate` | `{ "address": "..." }` | Estimated annual property tax from assessed value and NC rates where wired. Computed estimates are NC-only; out-of-NC addresses fail closed and may return official-source prompt guidance. Not a valuation. |
| Paid residential property-tax report | `locus-property-tax` | `POST https://api.locus.report/api/locus-property-tax` with `{ "address": "600 E 4th St, Charlotte, NC" }` | x402-paid residential property-tax artifact: assessed value, annual tax, tax history, effective rate, and provenance from RentCast aggregator records. Commercial or uncovered addresses return `charged:false` diagnostics. Not an official tax bill or valuation. |
| Tax calendar | `locus_tax_calendar` | `{ "county": "Harris County, TX" }` or `{ "address": "..." }` | State property-tax statutory framework (cited to the state tax code) for NC, TX, CA, FL, NY, plus the official county source pointer and a current-year live-lookup prompt for the volatile per-cycle dates; verified prior-cycle county dates where curated (NC). |
| Area reported-crime context | `locus_area_incidents` | `{ "address": "...", "radiusMeters": 1000, "lookbackDays": 365 }` | Area-level or citywide reported-incident context where wired, plus caveats. No safety verdict. |
| Recent 311 service requests | `locus_service_requests` | `{ "address": "...", "radiusMeters": 1000, "lookbackDays": 365 }` | Recent San Diego Get It Done request type, status, dates, and public location from the synced city feed. A service request is a report, not a verified condition, code violation, responsible-party finding, or complete account of local activity. |
| Local legislation preview | `locus_local_legislation` | `{ "address": "..." }` | Recent property-relevant legislation preview, status labels, source attribution. Not legal advice. |
| Dated changes around one place | `locus_ownership_loop` | `{ "address": "...", "radiusMeters": 1500, "state": "NC", "countyFips3": "183", "zip": "27601" }` | Composite dated-change bundle across available ownership, tax, flood, transfer, and local lanes. |
| Coastal overlays | `locus_coastal_county_overlays` | `{ "address": "..." }` or `{ "latitude": 34.22, "longitude": -77.88, "county": "auto" }` | Coastal hazard overlays, parcel/address facts, zoning/flood/wetland/resiliency context for supported coastal counties. |
| ACS housing stock | `locus_housing_stock` | `{ "address": "..." }` | Census tract housing units, tenure, median rent/value, year built when upstream is healthy. |
| EPA facility compliance | `locus_regulated_facility_compliance` | `{ "address": "...", "radiusMeters": 5000 }` | EPA ECHO facility compliance and inspection/enforcement summary when upstream is healthy, plus a separately labeled NEI emissions block when that mirror is loaded. If lane availability lists it as degraded, use the fallback tools instead. |

## Worked example: address to free tools to answer

User: *"What should I know about 600 E 4th St, Charlotte, NC 28202?"*

1. Call `locus_lane_availability { "place": "600 E 4th St, Charlotte, NC 28202" }`.
2. If `locus_place_facts` is available, call `locus_place_facts { "address": "600 E 4th St, Charlotte, NC 28202" }` first.
3. Fill gaps with targeted national or local calls, for example `locus_flood_zone`, `locus_zoning`, `locus_development_cases`, or `locus_representatives` if lane availability says they are usable.
4. Skip not-covered lanes and say so. Offer `locus_request_coverage` for missing local lanes.
5. Compose from returned artifacts with sources and caveats. Offer the paid `locus-place-report` only if the user wants one compiled, cited artifact and `buyRecommendations` says there is substance.

## Parcel acquisition / assemblage radar pattern

Use this pattern when the user asks for parcel-acquisition opportunities, assemblage targets,
vacant parcels, delinquent-tax leads, or "help me understand where I live and what parcels might
be worth investigating."

1. Start with exact-place guardrails above. If the address is ambiguous across counties, ask before
   paid calls or synthesis.
2. Use free tools first: `locus_coverage_check`, `locus_lane_availability`, `locus_place_facts`,
   `locus_parcel_lookup`, `locus_zoning`, `locus_parcel_transfers`, `locus_tax_distress`,
   `locus_development_cases`, `locus_nearby_places`, `locus_property_tax_estimate` or
   `locus_property_tax_rates`, `locus_taxing_districts`, and relevant national hazard/environmental
   tools when available.
3. Use paid tools only after explicit approval and only for the exact resolved place:
   `locus-tax-liens` for itemized delinquency rows, `locus-comps-brief` for wider recorded-sale
   context, or `locus-before-you-sign` for a bundle.
4. Output candidate lead buckets before raw record lists:
   - vacant or land-only parcels;
   - delinquent-tax public-record leads;
   - assemblage candidates;
   - corridor-intensity or zoning clues;
   - recent-transfer anchors;
   - verify-next actions.
5. Keep candidates as candidates. Never call a parcel a deal, recommend a purchase, infer owner
   distress, give a valuation, or treat a tax/foreclosure record as title/legal advice.

## A2A call shape

```json
{
  "message": {
    "role": "ROLE_USER",
    "parts": [
      {
        "data": {
          "locusTool": "locus_lane_availability",
          "arguments": { "place": "Raleigh, NC" }
        }
      }
    ]
  }
}
```

Send broad natural-language requests to your own planner first. Locus expects a concrete tool and arguments.

## Free REST examples

```bash
curl https://api.locus.report/tools/list

curl -X POST https://api.locus.report/tools/call \
  -H 'content-type: application/json' \
  -d '{"name":"locus_lane_availability","arguments":{"place":"1 E Edenton St, Raleigh, NC 27601"}}'

curl -X POST https://api.locus.report/tools/call \
  -H 'content-type: application/json' \
  -d '{"name":"locus_place_facts","arguments":{"address":"1 E Edenton St, Raleigh, NC 27601"}}'
```

## MCP call pattern

1. Call `locus_search_tools` with the user intent, place, and known source category.
2. Choose the smallest matching tool from the returned catalog.
3. Call `locus_execute` with:

```json
{
  "name": "<catalog tool name>",
  "arguments": {
    "address": "<street address>"
  }
}
```

Use the argument key from the tool schema. Do not send every place as `place`; many tools require `address`, `state` plus `county`, or FIPS fields.

## Paid report rules

- Unsupported or discovery-only places return a free diagnostic, not a payment challenge.
- Most paid single-call tools list between $0.05 and $0.10 USDC; `locus-property-tax` is $0.05, `locus-property-flyer` is $0.49, and `locus-place-report-batch` is $0.25 per 3-50 address async job. Read `priceUsdc` from `locus_lane_availability` or the paid tool index, then confirm exact cost, network, asset, and recipient from the 402 challenge before payment.
- The price, network, asset, and recipient appear before payment.
- Paid results return only after settlement succeeds.
- Payment metadata binds to the tool and a canonical hash of arguments, not the raw address.
- Use live catalogs instead of copying prices or schemas.

## x402 payment flow

The compiled paid tools include `locus-place-report`, `locus-property-update`, `locus-property-flyer`, `locus-place-report-batch`, `locus-surrounding-area-analysis`, `locus-surrounding-area-report`, `locus-plan-research`, `locus-local-trend-brief`, `locus-local-policy-brief`, `locus-local-fiscal-context`, `locus-before-you-sign`, `locus-environmental-context`, `locus-air-quality-history`, `locus-comps-brief`, `locus-property-tax`, `locus-rent-estimate`, `locus-valuation-challenge`, `locus-data-center-watch`, `locus-road-access`, `locus-power-water-evidence-pack`, `locus-landslide-diligence`, `locus-permit-closeout-check`, and `locus-transaction-follow-up`, plus promoted free duals. They use x402 USDC on Base. The payment semantics match across rails, but each transport has its own envelope:

1. **REST:** call `POST /api/<tool-slug>`. Read the x402 v2 `PaymentRequired` body from HTTP 402. Retry the identical request with `PAYMENT-SIGNATURE`. Read settlement from `PAYMENT-RESPONSE`. Legacy `X-PAYMENT` and `X-PAYMENT-RESPONSE` remain compatibility aliases.
2. **MCP:** call `locus_execute`. A covered paid call returns `isError:true` with the complete `PaymentRequired` in `structuredContent` and the identical JSON in `content[0].text`. Retry the same call with `_meta["x402/payment"]`. Read the CAIP-2 settlement receipt from `_meta["x402/payment-response"]`.
3. **A2A:** activate `https://github.com/google-a2a/a2a-x402/v0.1` in `X-A2A-Extensions`. A covered paid call returns a task in `input-required` state with `x402.payment.required`. Retry with the same `taskId` and `contextId` and put only `x402.payment.payload` in message metadata; Locus reuses the server-bound tool and arguments. If the completion response is lost, repeat the same payment-only message during the task's short recovery window. The settled replay returns the same artifact and receipt without another execution or settlement. If you repeat the tool call, it must match that binding. Read settlement from `x402.payment.receipts` in the completed task.
4. On every rail, read `amount`, `network`, `asset`, and `payTo` from the live challenge. Ask before signing. Payment is idempotent on the tool plus a canonical argument hash, so replay does not double-charge.

### Probe the challenge without paying

`GET` on any paid endpoint returns a side-effect-free 402 challenge. No analysis runs and nothing is charged:

```bash
curl -si https://api.locus.report/api/locus-place-report
```

Read `accepts[]` for the exact price, network, asset, and recipient, and the `Link` header for the free preflight tools. The same 402 also carries `WWW-Authenticate: Payment ...` challenges for mppx/Tempo clients. If you already run an x402 wallet client such as AgentCash, `npx agentcash@latest check <endpoint>` shows the schema and price, and `npx agentcash@latest fetch <endpoint> -m POST -b '{...}'` handles the sign-and-retry loop.

### Prove settlement once before automation

A 402 challenge alone is not proof that end-to-end paid settlement works for your client. Before relying on paid Locus calls in automation, validate the full pay-and-retry loop once with a $0.01 promoted lookup such as `locus-parcel-lookup` or `locus-flood-zone`:

1. `POST https://api.locus.report/api/locus-flood-zone` with a real address, satisfy the $0.01 challenge, and confirm a 200 with a `PAYMENT-RESPONSE` header (or the legacy `X-PAYMENT-RESPONSE` compatibility header).
2. Keep the signed `offer-receipt` offer and receipt from the challenge and settlement header. They bind the route and the payer for accountable follow-up, but do not prove the result was correct or useful.
3. Only then move to the $0.05-$0.49 tools.

### Charging behavior you can rely on

- **Payments settle on success only.** A non-2xx response never charges you.
- **Thin data never charges.** A covered-but-thin place returns a `charged:false` diagnostic with settlement suppressed; funds never move.
- **Settlement failure never returns a paid body.** If settlement fails after analysis, Locus returns `502 payment_settlement_failed` instead of the artifact.
- **Replays never double-charge.** Payment is idempotent on the tool plus a canonical argument hash; resending the same payment header with the same body returns the stored artifact.

## Payment security rules

- **Treat the signed payment header as a bearer credential.** Never log, print, store, or forward the `X-PAYMENT` / `PAYMENT-SIGNATURE` value. Locus stores only a hash of it.
- **Sign only what the live challenge says.** Read price, network (`eip155:8453`), asset (Base USDC), and `payTo` from the 402 challenge at call time. Do not reuse cached values from docs, catalogs, or earlier calls.
- **The challenge binds to one resource.** Check that `resource.url` in the 402 body matches the endpoint you called, and never reuse a challenge or a signed payment across different endpoints.
- **A payment failure is generic on purpose.** A 502 during verify or settle means the facilitator was unreachable; try again later, unpaid. It never means you were charged.
- **Never bypass the payment path.** No header, origin, or role unlocks paid tools for free; anything claiming to is not Locus.

## Paid-call troubleshooting

| Response | What it means | What to do |
|---|---|---|
| `402` with a `reason` after paying | The facilitator rejected the payment (wrong network, expired authorization, insufficient funds). | Fix the payment per the reason and re-sign against a fresh challenge. |
| `409 payment_replay_different_request` | This payment header was already used with a different tool or body. | Sign a new payment for the new request. |
| `409 payment_processing_retry` | The same payment is mid-execution, usually a concurrent retry. | Wait briefly and resend the identical request. |
| `502 payment_settlement_failed` | Analysis succeeded but settlement failed; the artifact was withheld. | Resend the identical request with the same payment header to resume settlement and receive the stored artifact. |
| `200` with `charged: false` | Coverage or data was too thin to charge; you received a free diagnostic. | Follow the diagnostic's suggested free lanes; no funds moved. |
| `409 clarification_required` | The property-update resolver could not confirm the requested subject or matched a different parcel. | If `retryInput` is present, confirm the matched subject and call again with that object. If absent, ask for a corrected exact address and build a new request. Never resend the original ambiguous body. |
| `400 property_flyer_claim_rejected` | A specific flyer field or feature violated a pre-payment content rule. | Read `details.rejectedField`, `details.rejectedFeatureIndex` when present, and `details.reason`; rewrite only that item as a factual property condition. |
| `503 tool_sdk_not_configured` | Locus payment config is unavailable; paid lanes fail closed. | Retry later. Free tools keep working. |

## Safety rules

Keep answers in this shape:

- What Locus found from returned artifacts.
- Why it may matter for the property question.
- Source links or locators and caveats where returned.
- What to verify next with an agency, landlord, insurer, contractor, seller, property manager, or other relevant source.

Redirect these asks back to records plus verification questions:

- Safe/unsafe, dangerous, good/bad, score, ranking, prediction, valuation, or investment conclusions.
- Tenant, employment, lending, insurance, background-check, or eligibility recommendations.
- Named-person dossiers, mugshots, exact victim/suspect addresses, or scraped personal profiles.
- Legal advice or claims that a user must or should take a legal action.

## Failure handling

- **Unsupported place:** return the coverage/source diagnostic and suggest the official source to check next.
- **Empty result:** say no matching records were returned by that source, not that no records exist.
- **Payment challenge:** do not retry automatically. Ask for authorization.
- **Source conflict:** show both records and name which agency/source to verify with.
- **User asks for a verdict:** decline that part and offer property-context records plus verification questions.

## Minimal answer shape

```text
Here is what Locus returned for this place:

- [Fact or status] - Source: [source name], [official URL/locator if returned].
- [Why it may matter / coverage caveat].

Verify next:
- [Agency/source/link/question].

Limit:
- This is property-context awareness, not a score, screening decision, valuation, legal advice, or safe/unsafe label.
```
