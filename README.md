# Locus Agent Skill

[![skills.sh](https://skills.sh/b/velinussage/locus-agent-skill)](https://skills.sh/velinussage/locus-agent-skill)

Install the Locus agent skill for cited property and local-government context over MCP, REST, A2A, free tools, and x402 paid reports.

Locus returns awareness and verification steps, not a verdict. Do not use it to score, rank, screen, value, predict, or label a person, property, block, or neighborhood as safe or unsafe.

## Install with GitHub CLI

```bash
gh skill install velinussage/locus-agent-skill locus-agent-tools
```

Preview first:

```bash
gh skill preview velinussage/locus-agent-skill locus-agent-tools
```

## Install with skills.sh

```bash
npx skills add velinussage/locus-agent-skill --skill locus-agent-tools
```

Use once without installing:

```bash
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

Docs: https://docs.locus.report/skill.md
