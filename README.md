# @velinussage/locus-agent-skill

[![skills.sh](https://skills.sh/b/velinussage/locus-agent-skill)](https://skills.sh/velinussage/locus-agent-skill)

Install two Locus agent skills: the general MCP/REST/A2A capability guide and the opt-in surrounding-area purchase workflow.

Locus returns awareness and verification steps, not a verdict. Do not use it to score, rank, screen, value, predict, or label a person, property, block, or neighborhood as safe or unsafe.

## Install with this package

```bash
npx @velinussage/locus-agent-skill add
```

By default this writes `locus-agent-tools/` and `locus-surrounding-area-analysis/` to `$CODEX_HOME/skills` or `~/.codex/skills`. The general public skill routes relevant requests to the purchase workflow; it does not turn ordinary Locus discovery into a paid flow.

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

- 23+ national free tools work for geocodable US addresses, with 51 free tools total in the live catalog.
- `locus_place_facts` is the best first call when supported.
- `locus_lane_availability` shows national, local, varies, not-covered, and degraded lanes before payment.
- Free tools stay read-only and cited.
- Paid endpoints use live x402 discovery and require explicit user authorization.
- The surrounding-area workflow requires two separate approvals: $0.10 for the reusable JSON packet, then $0.15 for an eligible HTML/PDF report within 24 hours. Packet and report retrieval are proof-gated for 90 days.

Docs: <https://docs.locus.report/skill.md>


## Paid endpoint surface

The live paid index is authoritative: <https://api.locus.report/.well-known/ai-tool/index.json>. Current endpoints are:

| Endpoint | Current price | Notes |
|---|---:|---|
| `POST /api/locus-place-report` | `$0.05` | Compiled cited property-context artifact. |
| `POST /api/locus-place-report-batch` | `$0.25` | Async 10-50 address portfolio job, one settlement. |
| `POST /api/locus-local-trend-brief` | `$0.05` | Permit, 311, and code-case local-change brief where source coverage is strong enough. |
| `POST /api/locus-local-policy-brief` | `$0.07` | Property-relevant bills, agendas, ordinances, tax, fee, bond, housing, and permit-change context. |
| `POST /api/locus-before-you-sign` | `$0.07` | Pre-decision bundle over parcel, trend, and policy components. |
| `POST /api/locus-environmental-context` | `$0.05` | Address-level EPA TRI/RCRA/SDWIS/radon context ranked by distance where possible. |
| `POST /api/locus-property-tax` | `$0.05` | Residential property-tax artifact with assessed value, annual tax, history, effective rate, and provenance. |
| `POST /api/locus-surrounding-area-analysis` | `$0.10` | Stored, bounded JSON evidence packet. Charge-suppressed unless the subject, surrounding-parcel foundation, and another component complete. |
| `POST /api/locus-surrounding-area-report` | `$0.15` | One HTML/PDF rendering of an eligible packet; no research rerun. |

Unsupported, discovery-only, commercial, or insufficient-data cases return free diagnostics instead of charging where applicable.

## License

MIT.
