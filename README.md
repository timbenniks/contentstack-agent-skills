# Contentstack Agent Skills

A collection of [Agent Skills](https://agentskills.io) that extend AI agents with specialized capabilities for working with Contentstack.

## Skills

| Skill | Description |
|---|---|
| [contentstack-cli-assistant](contentstack-cli-assistant/) | Intelligent CLI assistant that translates natural language into csdx commands. Covers auth, export/import, branches, publishing, migrations, launch, audit, and more. |

## Installation

### Prerequisites

- Node.js v20-22.x
- Contentstack CLI installed globally (`npm install -g @contentstack/cli`) or available via npx
- A Contentstack account with appropriate permissions

### Using with Claude Code

Add the skill to your project by copying the `contentstack-cli-assistant` folder into your repository, or install it directly:

```bash
npx @anthropic-ai/claude-code skill add https://github.com/timbenniks/contentstack-skills/tree/main/contentstack-cli-assistant
```

### Using with other agents

Copy the `contentstack-cli-assistant` folder into your agent's skill directory. The skill follows the open [Agent Skills](https://agentskills.io) format and can be used by any agent that supports it.

### Manual setup

1. Clone this repository:
   ```bash
   git clone https://github.com/timbenniks/contentstack-skills.git
   ```

2. Ensure the Contentstack CLI is available:
   ```bash
   # Install globally
   npm install -g @contentstack/cli

   # Or verify it works via npx
   npx @contentstack/cli --version
   ```

3. Point your agent to the `contentstack-cli-assistant/SKILL.md` file.

## What the CLI assistant does

The skill acts as an intelligent interface to the Contentstack CLI (`csdx`). Instead of memorizing 60+ commands with hundreds of flags, you describe what you want in plain language and the agent:

1. Figures out the right command(s)
2. Asks for any missing parameters
3. Validates prerequisites (logged in, correct region, tokens set up)
4. Shows a plan with the exact commands before running anything
5. Executes only after you confirm
6. Summarizes the results with suggested next steps

### Covered command areas

- **Authentication** - login, logout, token management, aliases
- **Configuration** - regions, rate limits, logging
- **Export** - full stack, specific modules, query-based, CSV
- **Import** - full stack, specific modules, replace/skip existing
- **Stack cloning** - structure only or with content
- **Branches** - create, delete, compare, merge
- **Bulk publish/unpublish** - entries, assets, filters, retry failed
- **Migration** - scripts, HTML RTE to JSON RTE, taxonomies, master locale
- **Audit** - identify and fix issues in exported data
- **Launch** - deploy projects, view logs, manage environments
- **Bootstrap/Seed** - starter apps, seed from GitHub repos

## What are Agent Skills?

Agent Skills are folders of instructions, scripts, and resources that agents can discover and use to do things more accurately and efficiently. They follow an open format specification that allows skills to be reused across different agent products.

Learn more at [agentskills.io](https://agentskills.io).
