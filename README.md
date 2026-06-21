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

- 23+ national free tools work for geocodable US addresses, with 50 free tools total in the live catalog.
- `locus_place_facts` is the best first call when supported.
- `locus_lane_availability` shows national, local, varies, not-covered, and degraded lanes before payment.
- Free tools stay read-only and cited.
- Paid tools use x402 and require explicit user authorization.

Docs: <https://docs.locus.report/skill.md>
