---
name: locus-agent-tools
preamble-tier: 1
version: 1.3.0
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

- **19 national tools work for any US address, for free, with no payment or coverage check needed.** Use them for rural addresses too, including flood zone, flood gauges, FEMA events, NOAA storms, radon, wildfire risk, cleanup sites, toxic releases, water systems, representatives, governing districts, fair-market rents, nearby places, ordinance leads, and data-center/source-discovery prompts.
- **National free tools cover all 50 states for geocodable US addresses.** Local lanes are wired jurisdiction by jurisdiction and are growing. Always expect national context. Treat local parcel, zoning, permit, tax, and development-case depth as coverage-dependent.
- **Start with `locus_place_facts` when lane availability says it is available.** It is the one-call address bundle for supported parcel areas: parcel facts, FEMA flood zone, governing districts, transportation context, and tax context where wired.
- **Use `locus_lane_availability` before paid calls.** It maps national, local, varies, not-covered, and degraded lanes, then gives per-paid-tool buy recommendations.
- **Do not memorize paid prices from this skill.** Call the live paid catalog or `locus_lane_availability` for current `priceUsdc`, then read the x402 challenge for exact price, chain, recipient, and schema before payment.

## Quick connect

Install this skill in Codex-style agents:

```bash
npx @velinussage/locus-agent-skill add
```

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

- [Agent Card](https://api.locus.report/.well-known/agent-card.json) - A2A skills and message endpoint.
- [Free tool catalog](https://api.locus.report/tools/list) - free tool schemas, descriptions, and read-only metadata.
- [Paid tool index](https://api.locus.report/.well-known/ai-tool/index.json) - current prices, schemas, and manifests.
- [MCP catalog](https://mcp.locus.report/catalog) - searchable tool catalog.
- [API base](https://api.locus.report) - health and discovery links.

The live catalogs are authoritative for tool names, schemas, prices, and endpoints. Do not copy stale tool definitions into prompts.

## Two coverage tools, know the difference

- **`locus_coverage_check { "place": "..." }`** asks whether Locus has source coverage for a jurisdiction at all. Use it for a broad city, county, ZIP, or address scope check.
- **`locus_lane_availability { "place": "..." }`** is the per-address capability map. It returns which exact tools are national, local, varies, not covered, or degraded, plus buy signals for paid tools. Call this before address-specific local lanes and before paying.
- **`locus_coverage_map {}`** returns the whole registry view. Use it when an agent needs breadth, not one address.

## Workflow

1. **Discover tools.** Use the table below for common intents. The live catalog, `locus_search_tools` over MCP or `GET /tools/list` over REST, is authoritative for the full current set and exact schemas.
2. **For a broad address question, call `locus_place_facts` first if available.** It often replaces several separate calls. If lane availability marks it not covered, fall back to national free tools.
3. **For local depth, check availability for the exact place.** Call `locus_lane_availability { "place": "<ZIP or address>" }`, then call only the lanes it marks `available`. For `varies`, call the free tool once and let its returned coverage/status decide.
4. **Run the smallest tool by intent.** Use `locus_execute` over MCP, `/a2a/v1/message:send` with a DataPart over A2A, or `POST /tools/call` over free REST. Prefer one targeted free tool over a paid bundle when the question is narrow.
5. **Ground every fact.** Answer only from returned artifacts. Include source names, links or locators, fetched timestamps where present, and caveats.
6. **Pay only on explicit authorization.** A paid tool returns an x402 challenge. Show price, chain, recipient, and tool, then retry only after the user approves.

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
      "degraded": [
        { "tool": "locus_housing_stock", "access": "free", "why": "upstream Census geocoder is failing" }
      ]
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
| Which tools will return data here? | `locus_lane_availability` | `{ "place": "600 E 4th St, Charlotte, NC" }` | Jurisdiction, `lanes.national/local/varies/notCovered/degraded`, paid buy signals, warnings. |
| Is this city/county/ZIP in source coverage? | `locus_coverage_check` | `{ "place": "Raleigh, NC" }` | Resolved jurisdiction, supported/partial/discovery status, verified sources, missing source gaps. |
| What is the whole coverage registry? | `locus_coverage_map` | `{}` | Registry-level coverage inventory for tools and jurisdictions. |
| Request coverage for a missing place | `locus_request_coverage` | `{ "place": "Park County, MT" }` | Acknowledgement and demand signal. No records. |
| Inspect official source cards | `locus_source_card_check` | `{ "place": "Raleigh, NC" }` or `{ "jurisdiction": "us-nc-raleigh" }` or `{ "cardId": "us-nc-durham:permits" }` | Source-card status, provenance, endpoint, verification method, timestamp. |
| Verify a citation URL | `locus_verify_citation` | `{ "sourceUrl": "https://...", "recordId": "optional", "jurisdiction": "optional" }` | Whether a citation matches a known source card, with provenance context. |
| What policy sources govern here? | `locus_policy_sources` | `{ "place": "Raleigh, NC" }` | State, county, city policy-source list, legal geographies, source links. |
| Read aggregate coverage demand | `locus_coverage_demand` | `{}` | Aggregate requested-coverage demand, not place records. |

### Best first call for supported address context

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| Give me one broad free snapshot for this address | `locus_place_facts` | `{ "address": "1 E Edenton St, Raleigh, NC 27601" }` | One-call bundle: parcel facts where wired, FEMA flood zone, governing districts, transportation context, property-tax context where wired, citations, and caveats. Start here when available. |
| Parcel facts only | `locus_parcel_lookup` | `{ "address": "123 Oak Park Dr, Cary, NC 27519" }` | PII-safe parcel or place facts such as land use, acreage, year built where present, derived county/municipality, provenance. |
| Multiple caller-supplied parcels | `locus_parcel_set` | `{ "addresses": ["addr 1", "addr 2"] }` | Normalized facts for up to 25 supplied addresses in an assemblage. |

### 19 national free content tools

These work for geocodable US addresses at no cost. Some accept address directly. Others need county, state, ZIP, or FIPS. Use `locus_lane_availability` or any geocoder/FIPS resolver to derive those arguments when needed.

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| FEMA flood zone at a point | `locus_flood_zone` | `{ "address": "600 E 4th St, Charlotte, NC" }` or `{ "latitude": 35.22, "longitude": -80.84 }` | Flood zone code, SFHA flag, panel or DFIRM identifiers when returned, plain-language zone context, provenance, verify-next steps. |
| Nearby flood gauges | `locus_flood_gauges` | `{ "address": "...", "radiusMeters": 20000 }` | Nearby USGS/NWS gauges, latest stage/flow observations when available, threshold metadata, source links. |
| Governing districts | `locus_governing_districts` | `{ "address": "..." }` | Congressional, state senate, state house, county, municipality, and other governing geographies when resolved. |
| Representatives | `locus_representatives` | `{ "address": "..." }` | Cited federal and state officials for the address, districts, party/office/contact links where available. |
| Wildfire risk | `locus_wildfire_risk` | `{ "address": "..." }` | FEMA NRI tract-level wildfire rating, score, expected annual loss fields where returned, provenance, caveats. |
| Environmental history bundle | `locus_environmental_history` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` | EPA TRI, RCRA, SDWIS water systems, radon zone, and event-history verify links. |
| Cleanup sites | `locus_cleanup_sites` | `{ "address": "...", "radiusMeters": 5000 }` | EPA Superfund, NPL, brownfield, or cleanup-site records nearby with program/status fields and official profile links. |
| Toxic releases | `locus_toxic_releases` | `{ "state": "MT", "county": "Park County" }` | EPA TRI facilities for the county, chemical-release reporting fields where available, profile links. |
| RCRA hazardous-waste handlers | `locus_rcra_handlers` | `{ "state": "MT", "countyFips3": "067" }` or `{ "state": "MT", "zip": "59047" }` | RCRAInfo handlers, generator status/activity fields, coordinates, ECHO verify links. |
| EPA radon zone | `locus_radon_zone` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` | EPA county radon zone 1/2/3, county basis, testing/disclosure caveat, provenance. |
| Public water systems | `locus_water_systems` | `{ "state": "MT", "county": "Park County" }` | Active public water systems serving the county, PWSIDs, SDWIS/ECHO links. |
| Drinking-water violations | `locus_drinking_water_violations` | `{ "address": "..." }` or `{ "state": "MT", "county": "Park County" }` or `{ "pwsids": ["MT0000000"] }` | SDWIS/ECHO violation and enforcement rows, system ids, caveats that this is not proof of tap-water quality. |
| NOAA storm events | `locus_storm_events` | `{ "stateFips": "30", "countyName": "Park", "countyFips3": "067" }` | NOAA Storm Events history, event type, dates, damage estimates, narratives, and source links. Use `stateFips`, not `state`. |
| FEMA disaster and NFIP history | `locus_fema_events` | `{ "address": "..." }` or `{ "state": "MT", "countyFips3": "067", "zip": "59047" }` | FEMA disaster declarations by county, NFIP claims by ZIP, preliminary FIRM panel context where available. |
| HUD fair-market rents | `locus_fair_market_rents` | `{ "stateFips": "30", "countyFips3": "067", "year": 2026 }` | HUD FMR values by bedroom count and county/FIPS basis. |
| Nearby places and amenities | `locus_nearby_places` | `{ "address": "...", "radiusMeters": 800 }` | OpenStreetMap nearby amenities/places with categories, distance, and OSM provenance. |
| Data-center or large-development watch prompts | `locus_data_center_watch` | `{ "address": "..." }` or `{ "state": "NC", "county": "Wake County", "municipality": "Raleigh" }` | Bounded official-source query pack and watch leads for large projects, data centers, utility/water/planning sources. |
| Local ordinance leads | `locus_ordinance_leads` | `{ "address": "...", "topics": ["short_term_rental", "adu"] }` or `{ "place": "Raleigh, NC" }` | Jurisdiction-locked official-source query pack for ordinance research. Leads only, not legal advice. |
| Coastal county catalog | `locus_coastal_county_catalog` | `{}` or `{ "county": "new-hanover-nc" }` | Catalog of supported coastal source packs, overlays, source leads, and limitations. |

Temporarily degraded national tools may appear in `lanes.degraded`, such as `locus_regulated_facility_compliance` or `locus_housing_stock`. If degraded, use the suggested fallback from lane availability.

### Local or coverage-dependent free tools

| The question | Tool | Exact arguments | What it returns |
|---|---|---|---|
| Zoning district and overlays | `locus_zoning` | `{ "address": "..." }` or `{ "latitude": 35.78, "longitude": -78.64 }` | Governing zoning district, overlays/planning context where wired, source links. Rich coverage only in wired jurisdictions. |
| Development or rezoning nearby | `locus_development_cases` | `{ "address": "...", "radiusMeters": 1500 }` | Nearby development/rezoning case records, statuses, dates, identifiers, citations where wired. |
| Public capital projects nearby | `locus_capital_projects` | `{ "address": "...", "radiusMeters": 1609 }` | Government capital projects, public works, assessments where wired, with project ids and source links. |
| Building permits in supported metros | `locus_metro_permits` | `{ "city": "chicago", "latitude": 41.878, "longitude": -87.629, "radiusMeters": 1000, "sinceDate": "2025-01-01" }` | Nearby issued permit records, permit number/type/status, issue date, reported cost, source URL. |
| Transportation projects and traffic counts | `locus_transportation_context` | `{ "address": "...", "radiusMeters": 2000 }` | State DOT funded projects, traffic-count stations, routes, statuses where wired. |
| Transit stops and routes | `locus_transit_context` | `{ "address": "...", "radiusMeters": 400 }` | Transit stops, routes, shelter/ADA fields, headways where supported. |
| Recent nearby parcel transfers | `locus_parcel_transfers` | `{ "address": "...", "radiusMeters": 1500, "monthsBack": 12 }` | Recorded sales/transfers near the point where parcel-sale sources are wired. |
| Property-tax rates | `locus_property_tax_rates` | `{ "place": "Wake County, NC" }` | Adopted rate components and jurisdiction basis where wired. |
| Property-tax estimate | `locus_property_tax_estimate` | `{ "address": "..." }` | Estimated annual property tax from assessed value and rates where wired. Not a valuation. |
| Tax calendar | `locus_tax_calendar` | `{ "county": "Wake" }` | Appeal/payment/revaluation calendar for supported counties. |
| Area reported-crime context | `locus_area_incidents` | `{ "address": "...", "radiusMeters": 1000, "lookbackDays": 365 }` | Area-level or citywide reported-incident context where wired, plus caveats. No safety verdict. |
| Local legislation preview | `locus_local_legislation` | `{ "address": "..." }` | Recent property-relevant legislation preview, status labels, source attribution. Not legal advice. |
| Dated changes around one place | `locus_ownership_loop` | `{ "address": "...", "radiusMeters": 1500, "state": "NC", "countyFips3": "183", "zip": "27601" }` | Composite dated-change bundle across available ownership, tax, flood, transfer, and local lanes. |
| Coastal overlays | `locus_coastal_county_overlays` | `{ "address": "..." }` or `{ "latitude": 34.22, "longitude": -77.88, "county": "auto" }` | Coastal hazard overlays, parcel/address facts, zoning/flood/wetland/resiliency context for supported coastal counties. |
| ACS housing stock | `locus_housing_stock` | `{ "address": "..." }` | Census tract housing units, tenure, median rent/value, year built when upstream is healthy. |
| EPA facility compliance | `locus_regulated_facility_compliance` | `{ "address": "...", "radiusMeters": 5000 }` | EPA ECHO facility compliance and inspection/enforcement summary when upstream is healthy. |

## Worked example: address to free tools to answer

User: *"What should I know about 600 E 4th St, Charlotte, NC 28202?"*

1. Call `locus_lane_availability { "place": "600 E 4th St, Charlotte, NC 28202" }`.
2. If `locus_place_facts` is available, call `locus_place_facts { "address": "600 E 4th St, Charlotte, NC 28202" }` first.
3. Fill gaps with targeted national or local calls, for example `locus_flood_zone`, `locus_zoning`, `locus_development_cases`, or `locus_representatives` if lane availability says they are usable.
4. Skip not-covered lanes and say so. Offer `locus_request_coverage` for missing local lanes.
5. Compose from returned artifacts with sources and caveats. Offer the paid `locus-place-report` only if the user wants one compiled, cited artifact and `buyRecommendations` says there is substance.

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
- Do not rely on remembered price ranges. Read `priceUsdc` from `locus_lane_availability` or the paid tool index, then confirm exact cost, network, asset, and recipient from the 402 challenge before payment.
- The price, network, asset, and recipient appear before payment.
- Paid results return only after settlement succeeds.
- Payment metadata binds to the tool and a canonical hash of arguments, not the raw address.
- Use live catalogs instead of copying prices or schemas.

## x402 payment flow

The compiled paid tools (`locus-place-report`, `locus-local-trend-brief`, `locus-local-policy-brief`, `locus-before-you-sign`, `locus-environmental-context`) are gated with x402 micropayments in USDC on Base. The flow is identical over REST, MCP, and A2A:

1. Call the paid tool without payment: `POST https://api.locus.report/api/<tool-slug>` over REST, `locus_execute` over MCP, or `/a2a/v1/message:send` over A2A. The free `/tools/call` facade is never paid.
2. A covered place returns **HTTP 402** with `{ "x402Version": 2, "accepts": [{ "scheme": "exact", "network", "maxAmountRequired", "payTo", "asset", "extra": { "assetTransferMethod": "eip3009" } }] }`. Read live values from the challenge.
3. Sign an EIP-3009 USDC authorization for `maxAmountRequired` to `payTo` on `network` using an x402 client such as `x402-fetch`, `x402-axios`, or the Coinbase x402 SDK. Send it in the **`X-PAYMENT`** header and resend the identical request.
4. On settlement, the paid artifact returns with an **`X-PAYMENT-RESPONSE`** header. Show price, network, and recipient. Only pay after user authorization. Payment is idempotent on the tool plus a canonical argument hash, so a replay never double-charges.

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
