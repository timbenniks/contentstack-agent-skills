# Contentstack Agent Skills

A collection of [Agent Skills](https://agentskills.io) that extend AI agents with specialized capabilities for working with Contentstack.

## Skills

| Skill | Description |
|---|---|
| [contentstack-cli-assistant](contentstack-cli-assistant/) | Intelligent CLI assistant that translates natural language into csdx commands. Covers auth, export/import, branches, publishing, migrations, launch, audit, and more. |
| [contentstack-delivery-sdk-assistant](contentstack-delivery-sdk-assistant/) | Translates natural language into Contentstack TypeScript Delivery SDK code. Covers initialization, queries, filtering, pagination, image transforms, live preview, sync, and type generation. |
| [contentstack-js-to-ts-migration](contentstack-js-to-ts-migration/) | Hands-on migration assistant for converting JavaScript Delivery SDK code to the TypeScript SDK. Analyzes code, identifies changes, generates migrated TypeScript with types. |

## Installation

### Prerequisites

- Node.js v20-22.x
- Contentstack CLI installed globally (`npm install -g @contentstack/cli`) or available via npx
- A Contentstack account with appropriate permissions

### Using with Claude Code

Add skills to your project by copying the skill folder into your repository, or install directly:

```bash
# CLI assistant
npx @anthropic-ai/claude-code skill add https://github.com/timbenniks/contentstack-skills/tree/main/contentstack-cli-assistant

# Delivery SDK assistant
npx @anthropic-ai/claude-code skill add https://github.com/timbenniks/contentstack-skills/tree/main/contentstack-delivery-sdk-assistant

# JS → TS SDK migration assistant
npx @anthropic-ai/claude-code skill add https://github.com/timbenniks/contentstack-skills/tree/main/contentstack-js-to-ts-migration
```

### Using with other agents

Copy the skill folder into your agent's skill directory. The skills follow the open [Agent Skills](https://agentskills.io) format and can be used by any agent that supports it.

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

## What the Delivery SDK assistant does

The skill acts as an intelligent code-generation assistant for the Contentstack TypeScript Delivery SDK (`@contentstack/delivery-sdk`). Instead of reading SDK docs to figure out method chains and query operators, you describe what you need in plain language and the agent:

1. Figures out the right SDK methods
2. Asks for any missing info (content type UIDs, field names, filter values)
3. Generates complete, type-safe TypeScript code with imports and interfaces
4. Explains the code and suggests improvements

### Covered SDK areas

- **Setup & initialization** - stack config, regions, branches, caching
- **Entry queries** - single fetch, multi-entry queries, all filter operators
- **Asset queries** - list and fetch assets with dimensions
- **Content type queries** - list and inspect content type schemas
- **Image transformations** - resize, crop, format, quality, blur, sharpen, overlays
- **Live preview** - preview token setup, SSR integration
- **Sync** - initial sync, delta updates, pagination tokens
- **Type generation** - TypeScript interfaces from content models
- **JS → TS migration** - side-by-side method mapping for SDK upgrades

## What the JS → TS migration assistant does

The skill is a hands-on migration assistant for converting Contentstack JavaScript Delivery SDK code (`contentstack`) to the TypeScript Delivery SDK (`@contentstack/delivery-sdk`). Instead of reading migration docs and manually finding every API change, you paste your JavaScript code and the agent:

1. Scans your code and identifies every SDK pattern that needs to change
2. Builds a tailored migration checklist based on your specific usage
3. Generates complete migrated TypeScript with imports, interfaces, and proper method chains
4. Highlights breaking changes and new features worth adopting

### Covered migration areas

- **Package & imports** - package swap, import statement changes
- **Initialization** - positional args to named object, snake_case to camelCase
- **Method names** - PascalCase to camelCase (.ContentType → .contentType, .Entry → .entry)
- **Query operators** - named methods to .where() with QueryOperation enum
- **Renamed methods** - .language() → .locale(), .ascending() → .orderByAscending()
- **Assets** - .Assets() → .asset() (plural to singular, new features)
- **Image transforms** - function-based to OOP ImageTransform class
- **Caching** - .setCachePolicy() to cacheOptions config
- **Sync API** - removed {init: true}, parameter name changes
- **Live preview** - config key casing changes
- **Removed APIs** - complete reference with replacements
- **Type generation** - TypeScript interfaces from untyped JS content models
- **Utils extraction** - built-in to separate @contentstack/utils package
- **Promise patterns** - .then(success, error) to async/await

## What are Agent Skills?

Agent Skills are folders of instructions, scripts, and resources that agents can discover and use to do things more accurately and efficiently. They follow an open format specification that allows skills to be reused across different agent products.

Learn more at [agentskills.io](https://agentskills.io).
