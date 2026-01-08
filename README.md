# Contentstack Agent Skills

> **⚠️ Experimental** - These skills are experimental and require the Contentstack MCP to be installed and configured.

A collection of [Agent Skills](https://agentskills.io) that extend AI agents with specialized capabilities for working with Contentstack. These skills provide procedural knowledge and workflows for common Contentstack operations, enabling agents to perform complex tasks more accurately and efficiently.

## What are Agent Skills?

Agent Skills are folders of instructions, scripts, and resources that agents can discover and use to do things more accurately and efficiently. They follow an open format specification that allows skills to be reused across different agent products.

Learn more at [agentskills.io](https://agentskills.io).

## Prerequisites

- **Contentstack MCP**: These skills require the Contentstack MCP (Model Context Protocol) server to be installed and configured with appropriate stack credentials.
- **Agent compatibility**: Your agent must support the Agent Skills format to use these skills.

## Available Skills

### 1. Contentstack Release Runbook

**Skill**: `contentstack-release-runbook`

Create, clone, populate, and deploy Contentstack releases using the Contentstack MCP (CMA). Use for launch coordination, scheduled publishing, release verification, and publish queue triage.

**Key capabilities**:

- Plan releases (dry run with validation)
- Create or clone releases
- Add items to releases
- Verify release contents
- Deploy immediately or schedule deployment
- Verify and triage publish queue after deploy

**Default mode**: Plan (read-only by default)

### 2. Contentstack Stack Inventory

**Skill**: `contentstack-stack-inventory`

Generate a complete inventory report for a Contentstack stack using the Contentstack MCP. Includes content types and schemas, global fields, languages/locales, branches and branch aliases, environments, taxonomies and term structure.

**Use cases**:

- Auditing a stack
- Onboarding to an existing implementation
- Preparing migrations
- Documenting the current information architecture

**Output**: Comprehensive Markdown report

### 3. Contentstack Taxonomy Audit

**Skill**: `contentstack-taxonomy-audit`

Audit Contentstack taxonomies and terms using the Contentstack MCP (CMA). Exports taxonomy JSON, computes health metrics (depth, orphans, heavy terms), and produces an action-oriented report.

**Key capabilities**:

- Taxonomy inventory with counts
- Export summaries (JSON export per taxonomy)
- Term-level metrics (root terms, depth, orphans, referenced terms)
- Health analysis and recommendations

**Default mode**: Audit (read-only by default)

## Installation

Each skill is available as a `.skill` file that can be installed in your agent environment. The skill folders contain the `SKILL.md` specification file that defines the skill's behavior.

1. Ensure the Contentstack MCP is installed and configured
2. Install the desired skill(s) in your agent environment
3. Configure your agent to discover and use the installed skills

## Usage

Once installed, agents can discover and use these skills automatically based on the task at hand. Each skill includes:

- **Goal**: What the skill accomplishes
- **Inputs**: Required and optional parameters
- **Procedure**: Step-by-step workflow
- **Tools**: Required MCP tools from Contentstack
- **Output**: Expected results and format

## Skill Structure

Each skill follows the Agent Skills specification:

```
skill-name/
  └── SKILL.md    # Skill specification with metadata, goals, and procedures
```

The `SKILL.md` file includes:

- Frontmatter metadata (name, description, compatibility)
- Goal and scope
- Input requirements
- Tool dependencies
- Step-by-step procedures
- Output format specifications

## Specification

These skills follow the [Agent Skills specification](https://agentskills.io/specification). For details on the format and how to integrate skills into your agent, see:

- [What are skills?](https://agentskills.io/what-are-skills)
- [Specification](https://agentskills.io/specification)
- [Integrate skills](https://agentskills.io/integrate-skills)

## Contributing

These skills are experimental and may evolve based on feedback and usage. Contributions and improvements are welcome.

## License

See individual skill directories for license information.

## Related Links

- [Agent Skills](https://agentskills.io) - Learn more about the Agent Skills format
- [Contentstack MCP](https://github.com/contentstack/mcp-server-contentstack) - Contentstack MCP server documentation
- [Contentstack Documentation](https://www.contentstack.com/docs) - Official Contentstack documentation
