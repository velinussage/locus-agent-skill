# @velinussage/locus-agent-skill

[![skills.sh](https://skills.sh/b/velinussage/locus-agent-skill)](https://skills.sh/velinussage/locus-agent-skill)

Locus is the public record for any US address, built for AI agents. Ask about a place and it returns what official sources say about it: the parcel, flood maps, zoning, permits, taxes, sales, environmental filings, local legislation, and more. Every fact is cited to the agency that published it.

These fifteen skills teach an agent to use that record for the moments people actually face with a property: buying, comparing finalist homes, screening acquisition candidates, selling, owning, appealing a tax bill, handling an escrow jump or an insurance nonrenewal, a water leak, a missed payment, an inherited home, an aging parent's home, planning raw land, or working out where a photo was taken. The general skill finds the right tools for any question. The workflow skills carry one task from the first question to a cited, printable result, and ask before any paid step.

## Install with this package

```bash
npx @velinussage/locus-agent-skill add
```

By default this writes `locus-agent-tools/`, `locus-media-to-market/`, `locus-surrounding-area-analysis/`, `locus-property-tax-protest/`, `locus-land-entitlement/`, `locus-acquisition-radar/`, `compare-homes/`, `locus-home-watch/`, `locus-water-incident/`, `locus-seller-readiness/`, `locus-escrow-shock/`, `locus-payment-hardship/`, `locus-insurance-nonrenewal/`, `locus-inherited-home/` and `locus-aging-owner/` to `$CODEX_HOME/skills` or `~/.codex/skills`. The general public skill routes relevant requests to the specialist workflows; it does not turn ordinary Locus discovery into a paid flow.

Options:

```bash
npx @velinussage/locus-agent-skill add --target ./skills
npx @velinussage/locus-agent-skill add --target ./skills --force
npx @velinussage/locus-agent-skill --print
```

## Install with GitHub CLI

```bash
gh skill install velinussage/locus-agent-skill locus-agent-tools
```

Preview first:

```bash
gh skill preview velinussage/locus-agent-skill locus-agent-tools
```

## Install with skills.sh

The same skill is also available from the public GitHub repo through the open `skills` CLI:

```bash
npx skills add velinussage/locus-agent-skill --skill locus-agent-tools
```

Useful variants:

```bash
npx skills add velinussage/locus-agent-skill --skill locus-agent-tools -g
npx skills add velinussage/locus-agent-skill --skill locus-agent-tools -a claude-code -a codex
npx skills use velinussage/locus-agent-skill --skill locus-agent-tools
```

## Connect to Locus MCP

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

## What the skills teach agents

- Read the live free catalog for current tools and national lanes; rollout configuration can change the visible total.
- `locus_place_facts` is the best first call when supported.
- When you have only 1–5 photos or a short video clip and no street address yet, start with `locus_media_upload_init` and `locus_media_locate` (free EXIF/text check) or `locus-media-locate` ($0.05) / `locus-video-locate` ($0.10) / `locus-media-to-market` ($0.15) / `locus-video-to-market` ($0.25) to triangulate the region and follow its pre-populated `nextTools` (`livingThere`, `buyingThere`, `whatChangedNearby`).
- `locus_lane_availability` shows national, local, varies, not-covered, and degraded lanes before payment. With `forTool: "locus-before-you-sign-dossier"` it also returns a free `deliverablePreview`: which of the dossier's lanes are covered at the address and whether its charge rule is likely to be met.
- Paid deliverables charge on a published rule and return `chargeBasis` on every result, paid or not: the rule, which lanes returned cited records, which were checked with none, and which were unavailable. The before-you-sign dossier also takes `requireLanes`, so the buyer can make any lane a condition of payment.
- Free tools stay read-only and cited.
- The payment-hardship workflow routes mortgage trouble to HUD counseling agencies first, then official state foreclosure pointers and servicer fact cards; the tax branch checks payment status, calendars and relief-program pointers without an eligibility decision.
- `locus_rental_registration_check` declares its scope per jurisdiction. Long-term registration is wired in Minneapolis, Montgomery County, New York City, and Seattle; Denver is short-term-license only; Kansas City is building-enforcement only. It returns no legal-rental or compliance verdict.
- The property-tax protest workflow keeps deadline, filing-guide, evidence, browser handoff, and published-outcome facts separate from the owner's own filing and value position.
- The home-watch workflow sets up deadline and change events for an address or a signed-in pool, reads an empty poll as "nothing new seen", and offers each paid next step only with the owner's approval.
- The water-incident workflow keeps what failed, who maintains each pipe segment, and the resulting damage or bill separate, and gives insurer questions without a coverage verdict.
- Paid endpoints use live x402 and MPP discovery: x402 may offer USDC on Base and Solana (`accepts[]`), and MPP offers `USDC.e` (`0x20C000000000000000000000b9537d11c60E8b50`) on Tempo mainnet (`eip155:4217`, `chainId: 4217`) and, on the paid deliverables and deal analysis, Stripe card or Link, via `WWW-Authenticate: Payment`, all requiring explicit user authorization. Agents choose one complete offer (for example `-p x402 --payment-network base` or `-p mpp --payment-network tempo`). `locus-deal-analysis` offers Stripe and x402 only.
- The buyer workflows route to the $1.19 before-you-sign dossier (preview it free first) and, for a deal or a side-by-side of up to three addresses, to the $4.99 deal analysis: model-written prose over cited records, with every figure and citation checked by code and no buy-or-pass call.
- Reports and composed comparisons accept `brand` (brokerage name, agent, colors, logo) and return a ready-to-send `emailPackage`; the brand never changes the cited report itself.
- The surrounding-area workflow requires two separate approvals: $0.10 for the reusable JSON packet, then $0.15 for an eligible HTML/PDF report within 24 hours. Packet and report retrieval are proof-gated for 90 days.

Docs: <https://docs.locus.report/skill.md>


## Paid endpoint surface

The live paid index is authoritative: <https://api.locus.report/.well-known/ai-tool/index.json>. Selected endpoints are:

| Endpoint | Current price | Notes |
|---|---:|---|
| `POST /api/locus-place-report` | `$0.05` | Compiled cited property-context artifact. |
| `POST /api/locus-place-report-batch` | `$0.25` | Async 3-50 address portfolio job, one settlement. |
| `POST /api/locus-local-trend-brief` | `$0.05` | Permit, 311, and code-case local-change brief where source coverage is strong enough. |
| `POST /api/locus-local-policy-brief` | `$0.07` | Property-relevant bills, agendas, ordinances, tax, fee, bond, housing, and permit-change context. |
| `POST /api/locus-before-you-sign` | `$0.07` | Pre-decision bundle over parcel, trend, and policy components. |
| `POST /api/locus-environmental-context` | `$0.05` | Address-level EPA TRI/RCRA/SDWIS/radon context ranked by distance where possible. |
| `POST /api/locus-property-tax` | `$0.05` | Residential property-tax artifact with assessed value, annual tax, history, effective rate, and provenance. |
| `POST /api/locus-rental-registration-check` | `$0.01` | Paid dual of the free exact-building long-term-registration, short-term-license, or building-enforcement lookup, depending on jurisdiction. |
| `POST /api/locus-nyc-recording-history` | `$0.01` | Paid dual of the free bounded NYC ACRIS document index. No party or unit data and no title conclusion. |
| `POST /api/locus-renovation-site-context` | `$0.10` | Fixed renovation evidence workflow over parcel, permit, soil, terrain, flood, wetland, and radon context. |
| `POST /api/locus-solar-property-screen` | `$0.49` | Rollout-gated private one-address roof-solar and historical financial-reference screen. |
| `POST /api/locus-large-site-satellite-change` | `$0.15` | Direct Copernicus REST-only pair of dated Sentinel-2 Level-2A images for a 1-100 square-kilometre site. No automated change or condition claim. |
| `POST /api/locus-surrounding-area-analysis` | `$0.10` | Stored, bounded JSON evidence packet. Charge-suppressed unless the subject, surrounding-parcel foundation, and another component complete. |
| `POST /api/locus-surrounding-area-report` | `$0.15` | One HTML/PDF rendering of an eligible packet; no research rerun. |
| `POST /api/locus-before-you-sign-dossier` | `$1.19` | Homebuyer dossier over 11 lanes (parcel, flood, surrounding ring, zoning, recorded sales, EPA, legislation, permits, tax rate, HOA signs, NFIP cost) plus print-ready HTML. Charged only when parcel and flood plus 4 of the other 9 return cited records; `requireLanes` adds conditions. |
| `POST /api/locus-property-tax-protest-packet` | `$0.89` | Protest evidence packet. Charged only when the assessment position plus 2 of recorded sales, tax calendar, and appeal outcomes return cited records. |
| `POST /api/locus-escrow-insurance-dossier` | `$0.89` | Escrow-shock and insurance-nonrenewal dossier. Charged only when the parcel plus 3 of 5 owner lanes return cited records; federal RESPA text never counts. |
| `POST /api/locus-seller-readiness-packet` | `$0.85` | Pre-listing permit closeout packet. Charged only when the parcel and permit closeout check plus 1 more place-specific lane return cited records. |
| `POST /api/locus-deal-analysis` | `$4.99` | Cited deal brief or side-by-side of up to three addresses: scenario arithmetic, risks, and a diligence checklist. Stripe card/Link or x402; allow up to 240 seconds. |
| `POST /api/locus-surface-heat` | `$0.05` | Up to four recent clear summer Landsat land surface temperature readings, each cited to its scene. Ground surface, not air temperature. |
| `POST /api/locus-surface-heat-history` | `$0.25` | Summer land surface temperature by year since 2013, with scene IDs and the first-to-last change. |
| `POST /api/locus-field-water-use` | `$0.99` | OpenET satellite evapotranspiration by month and year for the last ten years, for farmland and rural-land diligence. An estimate, not a metered withdrawal. |
| Six paid-only atomic REST routes | `$0.01` each | Workplace jobs, Wikipedia, Commons metadata, PFAS, NEI, and electricity. No free, MCP, or A2A counterpart. |
| `locus-evaluation-packet` paid-only atomic | `$0.35` | Lender evaluation-support packet. No free, MCP, or A2A counterpart. |

Unsupported, discovery-only, commercial, or insufficient-data cases return free diagnostics instead of charging where applicable. The surface heat, heat history, and field water use routes are not charged when no clear reading or no data is found.

## License

MIT.
