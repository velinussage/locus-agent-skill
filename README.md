# @velinussage/locus-agent-skill

[![skills.sh](https://skills.sh/b/velinussage/locus-agent-skill)](https://skills.sh/velinussage/locus-agent-skill)

Install the Locus agent skill for agents that need cited property and local-government context over MCP, REST, A2A, free tools, and x402 paid reports.

Locus returns awareness and verification steps, not a verdict. Do not use it to score, rank, screen, value, predict, or label a person, property, block, or neighborhood as safe or unsafe.

## Install with this package

```bash
npx @velinussage/locus-agent-skill add
```

By default this writes `locus-agent-tools/SKILL.md` to `$CODEX_HOME/skills` or `~/.codex/skills`.

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

## What the skill teaches agents

- 23+ national free tools work for geocodable US addresses, with 51 free tools total in the live catalog.
- `locus_place_facts` is the best first call when supported.
- `locus_lane_availability` shows national, local, varies, not-covered, and degraded lanes before payment.
- Free tools stay read-only and cited.
- Seven paid endpoints use x402 and require explicit user authorization: place report, 10-50 address batch, trend brief, policy brief, before-you-sign bundle, environmental context, and residential property tax.

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
| `POST /api/locus-property-tax` | `$0.49` | Residential property-tax artifact with assessed value, annual tax, history, effective rate, and provenance. |

Unsupported, discovery-only, commercial, or insufficient-data cases return free diagnostics instead of charging where applicable.

## License

MIT.
