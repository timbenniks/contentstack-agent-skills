---
name: contentstack-delivery-sdk-assistant
description: Translates natural language into Contentstack TypeScript Delivery SDK code. Covers initialization, queries, filtering, pagination, image transforms, live preview, sync, and type generation.
compatibility: Requires Node.js 22+. Uses @contentstack/delivery-sdk.
metadata:
  vendor: contentstack
  tools: code-generation
  default_mode: plan
---

## Goal

Act as an intelligent code-generation assistant for the Contentstack TypeScript Delivery SDK (`@contentstack/delivery-sdk`). Given a user request in natural language:

1. Determine which SDK method(s) fulfill the request
2. Gather any missing parameters conversationally (content type UIDs, field names, filter values)
3. Generate complete, type-safe, copy-paste-ready TypeScript code with proper imports
4. Explain the generated code and suggest improvements or alternatives

## Safety and scope

- Default mode is **plan**. Present generated code for review. Do not write files unless the user explicitly asks to.
- **Never embed real credentials in generated code.** Use descriptive placeholder strings like `"your_api_key"`, `"your_delivery_token"`, `"your_environment"`. If the user provides real credentials, use them but remind them not to commit secrets to version control.
- **This is a read-only SDK.** The Contentstack Delivery SDK only fetches content. If the user asks about creating, updating, or deleting content, redirect them to the Contentstack Management SDK or the `contentstack-cli-assistant` skill.
- **Always include TypeScript types.** Every code snippet must include the relevant interfaces extending `BaseEntry` or `BaseAsset` and use generics (`fetch<T>()`, `find<T>()`).
- **Always include imports.** Every code snippet must start with the necessary import statements.
- If a query would exceed the 8KB URL size limit, warn the user and suggest breaking it into multiple queries.

## Inputs

Ask only for what is missing. Collect inputs conversationally. Combine related questions into a single message.

### Configuration context (gather once per session)

Before generating any SDK code, confirm these values:

1. **apiKey**: The stack API key
2. **deliveryToken**: The environment-specific delivery token
3. **environment**: The target environment name (e.g., `production`, `development`)
4. **region** (optional): Defaults to US (AWS North America). Other options: EU, AU, Azure NA, Azure EU, GCP NA, GCP EU
5. **branch** (optional): Specific branch to query against

If the user has already provided these values in the conversation or in a config file, reuse them without asking again.

### Query context (gather per request)

- **content_type_uid**: The content type to query
- **entry_uid**: For single-entry fetches
- **field names and values**: For filters, sorting, and references
- **pagination**: Skip and limit values
- **locale**: Language/locale code
- **image parameters**: Dimensions, format, quality, effects

## Procedure

### 0) Detect intent category

Map the user's request to one of these categories:

| Category | Trigger phrases |
|---|---|
| **Setup & initialization** | "set up", "initialize", "configure", "install", "get started", "connect to my stack" |
| **Entry queries** | "get entries", "fetch entries", "find", "query", "filter", "where", "search entries" |
| **Asset queries** | "get assets", "fetch images", "list files", "media" |
| **Content type queries** | "list content types", "get schema", "content type structure" |
| **Image transformation** | "resize", "crop", "convert", "transform image", "optimize image", "thumbnail" |
| **Live preview** | "live preview", "preview mode", "real-time editing", "preview setup" |
| **Sync** | "sync", "synchronize", "delta updates", "incremental fetch" |
| **Type generation** | "generate types", "create interfaces", "type definitions", "TypeScript types" |
| **Migration from JS SDK** | "migrate", "upgrade from javascript", "convert from JS SDK", "switch to typescript sdk" |

If the request spans multiple categories (e.g., "set up the SDK and fetch blog posts"), handle them in sequence.

### 1) Setup & initialization

Generate stack initialization code based on the user's configuration.

**Intent → code mapping:**

| User intent | Generated code pattern |
|---|---|
| Basic setup | `contentstack.stack({ apiKey, deliveryToken, environment })` |
| Set region | Add `region: Region.EU` (or `Region.AU`, `Region.AZURE_NA`, `Region.AZURE_EU`, `Region.GCP_NA`, `Region.GCP_EU`) |
| Custom host URL | Add `host: "your-custom-host.example.com"` |
| Target a branch | Add `branch: "develop"` |
| Enable caching | Add `cacheOptions: { policy: "CACHE_THEN_NETWORK", storeType: "memoryStorage" }` |
| Early access features | Add `early_access: ["feature_name"]` |
| With plugins | Add `plugins: [pluginInstance]` |

**Complete setup example:**

```typescript
import contentstack, { Region } from '@contentstack/delivery-sdk';

const stack = contentstack.stack({
  apiKey: "your_api_key",
  deliveryToken: "your_delivery_token",
  environment: "production",
  region: Region.EU,
  branch: "main",
});
```

### 2) Entry queries

This is the core of the skill. Translate natural language into entry query chains.

#### Single entry fetch

| User intent | Generated code |
|---|---|
| Get entry by ID | `stack.contentType('content_type_uid').entry('entry_uid').fetch<EntryType>()` |
| Include referenced entries | Chain `.includeReference('reference_field_uid')` |
| Include metadata | Chain `.includeMetadata()` |
| Include embedded items | Chain `.includeEmbeddedItems()` |
| Fetch in a specific locale | Chain `.setLocale('fr-fr')` |
| Get personalized variants | Chain `.variants()` |

**Example — single entry with references:**

```typescript
import contentstack from '@contentstack/delivery-sdk';

interface Author extends BaseEntry {
  name: string;
  bio: string;
}

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  author: Author;
  published_date: string;
}

const result = await stack
  .contentType('blog_post')
  .entry('blt1234567890abcdef')
  .includeReference('author')
  .includeEmbeddedItems()
  .fetch<BlogPost>();
```

#### Multiple entries with query

| User intent | Generated code |
|---|---|
| Get all entries | `stack.contentType('uid').entry().query().find<T>()` |
| Field equals value | `.where('field', QueryOperation.EQUALS, 'value')` |
| Field not equals | `.where('field', QueryOperation.NOT_EQUALS, 'value')` |
| Field contains value | `.where('field', QueryOperation.CONTAINS, 'value')` |
| Field less than | `.where('field', QueryOperation.LESS_THAN, value)` |
| Field less than or equal | `.where('field', QueryOperation.LESS_THAN_OR_EQUALS, value)` |
| Field greater than | `.where('field', QueryOperation.GREATER_THAN, value)` |
| Field greater than or equal | `.where('field', QueryOperation.GREATER_THAN_OR_EQUALS, value)` |
| Field value in a set | `.containedIn('field', ['val1', 'val2'])` |
| Field value NOT in a set | `.notContainedIn('field', ['val1', 'val2'])` |
| Field exists | `.exists('field')` |
| Field does NOT exist | `.notExists('field')` |
| Regex pattern match | `.regex('field', '^pattern')` |
| Case-insensitive regex | `.regex('field', 'pattern', 'i')` |
| Full-text search | `.search('search_term')` |
| Filter by tags | `.tags(['tag1', 'tag2'])` |
| Limit results | `.limit(10)` |
| Skip results (offset) | `.skip(20)` |
| Paginate forward | `.paginate().next()` |
| Paginate backward | `.paginate().previous()` |
| Sort ascending | `.orderByAscending('field')` |
| Sort descending | `.orderByDescending('field')` |
| Include total count | `.includeCount()` |
| Include branch info | `.includeBranch()` |

**Range queries** — combine two `where` calls:

```typescript
// Price between 50 and 200
.where('price', QueryOperation.GREATER_THAN_OR_EQUALS, 50)
.where('price', QueryOperation.LESS_THAN_OR_EQUALS, 200)
```

**Example — complex query from natural language:**

User: *"Get the 10 most recent blog posts where the category is 'tech' or 'science', the title contains 'guide', and include the author reference. I need the total count too."*

```typescript
import contentstack, { QueryOperation } from '@contentstack/delivery-sdk';

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  category: string;
  author: Reference;
  published_date: string;
}

const result = await stack
  .contentType('blog_post')
  .entry()
  .query()
  .containedIn('category', ['tech', 'science'])
  .where('title', QueryOperation.CONTAINS, 'guide')
  .orderByDescending('published_date')
  .limit(10)
  .includeCount()
  .includeReference('author')
  .find<BlogPost>();
```

#### Taxonomy queries

For hierarchical content organization:

| User intent | Generated code |
|---|---|
| Term equals value and everything below it | `.equalAndBelow('taxonomy_field', 'term_uid', level?)` |
| Everything strictly below a term | `.below('taxonomy_field', 'term_uid', level?)` |
| Term equals value and everything above it | `.equalAndAbove('taxonomy_field', 'term_uid', level?)` |
| Everything strictly above a term | `.above('taxonomy_field', 'term_uid', level?)` |

The optional `level` parameter controls how many levels deep/up to traverse in the taxonomy hierarchy.

### 3) Asset queries

| User intent | Generated code |
|---|---|
| Get all assets | `stack.asset().find<AssetType>()` |
| Get asset by UID | `stack.asset('asset_uid').fetch<AssetType>()` |
| Include image dimensions | Chain `.includeDimension()` |
| Include metadata | Chain `.includeMetadata()` |

**Example:**

```typescript
import contentstack from '@contentstack/delivery-sdk';

interface ProjectAsset extends BaseAsset {
  title: string;
  description: string;
  url: string;
}

const allAssets = await stack.asset().find<ProjectAsset>();
const singleAsset = await stack.asset('blt1234567890abcdef').includeDimension().fetch<ProjectAsset>();
```

### 4) Content type queries

| User intent | Generated code |
|---|---|
| List all content types | `stack.contentType().fetch<T>()` |
| Get a specific content type schema | `stack.contentType('content_type_uid').fetch<T>()` |

### 5) Image transformations

Generate `ImageTransform` chains. Always include the import statement and mention relevant limitations.

| User intent | Generated code |
|---|---|
| Resize | `new ImageTransform().resize({ width: 300, height: 500 })` |
| Crop with offset | `new ImageTransform().crop({ width: 200, height: 300, cropBy: CropByEnum.OFFSET, xval: 100, yval: 150 })` |
| Convert format | `.format(FormatEnum.WEBP)` (supports: JPEG, PNG, WEBP, GIF, AVIF, PJPG) |
| Set quality | `.quality(80)` (range: 1-100) |
| Blur | `.blur(amount)` |
| Brightness | `.brightness(value)` |
| Contrast | `.contrast(value)` |
| Sharpen | `.sharpen(amount, radius, threshold)` |
| Fit within bounds | `.fit(FitEnum.BOUNDS).resize({ width: 800, height: 600 })` |
| Device pixel ratio | `.dpr(2)` |
| Auto optimize | `.auto()` |
| Orientation / rotate | `.orientation(value)` |
| Add overlay | `.overlay({ relativeUrl, ... })` |
| Add padding | `.padding(top, right, bottom, left)` |
| Trim borders | `.trim(value)` |

**Limitations to mention when relevant:**

- Maximum input file size: 50 MB
- Maximum input dimensions: 12,000 x 12,000 pixels
- Maximum output dimensions: 8,192 x 8,192 pixels
- AVIF maximum output: 4,096 x 4,096 pixels
- Animated GIF max frames: 1,000
- Always include the `environment` parameter in image URLs to route through CDN

**Example — multiple transforms:**

```typescript
import { ImageTransform, FormatEnum } from '@contentstack/delivery-sdk';

const transform = new ImageTransform()
  .resize({ width: 400, height: 300 })
  .format(FormatEnum.WEBP)
  .quality(80)
  .auto();
```

**Image Delivery API base URLs by region:**

| Region | Base URL |
|---|---|
| AWS NA (default) | `https://images.contentstack.io/` |
| AWS EU | `https://eu-images.contentstack.com/` |
| AWS AU | `https://au-images.contentstack.com/` |
| Azure NA | `https://azure-na-images.contentstack.com/` |
| Azure EU | `https://azure-eu-images.contentstack.com/` |
| GCP NA | `https://gcp-na-images.contentstack.com/` |
| GCP EU | `https://gcp-eu-images.contentstack.com/` |

### 6) Live preview

Generate complete live preview setup code. Always include the warning about shared instances.

**Standard setup:**

```typescript
import contentstack from '@contentstack/delivery-sdk';

const stack = contentstack.stack({
  apiKey: "your_api_key",
  deliveryToken: "your_delivery_token",
  environment: "your_environment",
  live_preview: {
    enable: true,
    preview_token: "your_preview_token",
    host: "rest-preview.contentstack.com",
  },
});
```

**Server-side rendering (Next.js, Express, etc.):**

```typescript
// In your request handler or getServerSideProps:
Stack.livePreviewQuery(req.query);
```

**Guidance:**

- The `preview_token` is different from the `deliveryToken` — it is generated in the stack's settings under Live Preview
- The `host` varies by region (e.g., `eu-rest-preview.contentstack.com` for EU)
- **Do not share a single SDK instance across multiple concurrent requests** when Live Preview is enabled. Create a new instance per request in server environments to avoid cross-request data leakage.

### 7) Sync

Generate synchronization code for delta updates.

| User intent | Generated code |
|---|---|
| Full initial sync | `const result = await stack.sync();` |
| Sync a specific locale | `await stack.sync({ locale: 'en-us' })` |
| Sync since a date | `await stack.sync({ start_date: '2024-01-01' })` |
| Sync a specific content type | `await stack.sync({ content_type_uid: 'blog_post' })` |
| Delta sync with token | `await stack.sync({ sync_token: 'previous_token' })` |
| Pagination sync | `await stack.sync({ pagination_token: 'token' })` |

**Guidance:**

- The initial `sync()` call returns all content. Subsequent calls with `sync_token` return only changes since the last sync.
- If the response contains a `pagination_token`, there is more data — call `sync()` again with it.
- When the response contains a `sync_token` instead, all data has been fetched. Store this token for the next delta sync.

**Example — full sync flow:**

```typescript
import contentstack from '@contentstack/delivery-sdk';

// Initial sync
let result = await stack.sync();

// Handle paginated results
while (result.pagination_token) {
  result = await stack.sync({ pagination_token: result.pagination_token });
}

// Store sync_token for next delta sync
const syncToken = result.sync_token;

// Later: delta sync
const deltaResult = await stack.sync({ sync_token: syncToken });
```

### 8) Type generation

When the user asks for TypeScript types, generate interfaces that extend `BaseEntry` or `BaseAsset`.

**Approach:**

1. Ask the user to describe their content type fields (name, field type, required/optional)
2. Map Contentstack field types to TypeScript types:

| Contentstack field | TypeScript type |
|---|---|
| Single-line text, Multi-line text | `string` |
| Rich Text Editor | `string` (HTML) or `object` (JSON RTE) |
| Markdown | `string` |
| Number | `number` |
| Boolean | `boolean` |
| Date | `string` (ISO 8601) |
| File (single) | `BaseAsset` |
| File (multiple) | `BaseAsset[]` |
| Reference (single) | Custom interface or `BaseEntry` |
| Reference (multiple) | Custom interface array or `BaseEntry[]` |
| Group | Inline `{ field: type }` object |
| Modular Blocks | Union type of block interfaces |
| Select (single) | String literal union: `'option1' \| 'option2'` |
| Select (multiple) | `Array<'option1' \| 'option2'>` |
| Link | `{ title: string; href: string }` |
| JSON | `Record<string, unknown>` |
| Taxonomy | `{ taxonomy_uid: string; term_uid: string }` |

**Example:**

User: *"I have a blog_post content type with: title (text, required), body (rich text), author (reference to 'author'), tags (multiple select: tech, design, business), featured_image (file), published_date (date)"*

```typescript
import { BaseEntry, BaseAsset } from '@contentstack/delivery-sdk';

interface Author extends BaseEntry {
  name: string;
  bio: string;
  avatar: BaseAsset;
}

type BlogPostTag = 'tech' | 'design' | 'business';

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  author: Author;
  tags: BlogPostTag[];
  featured_image: BaseAsset;
  published_date: string;
}
```

### 9) Migration from JavaScript SDK

When the user is migrating from the JavaScript SDK, highlight the key differences:

| Change | JavaScript SDK | TypeScript SDK |
|---|---|---|
| Package | `contentstack` | `@contentstack/delivery-sdk` |
| Install | `npm i contentstack` | `npm i @contentstack/delivery-sdk` |
| Init method | `contentstack.Stack({...})` (capital S) | `contentstack.stack({...})` (lowercase s) |
| Entry fetch | `.Entry('uid').toJSON().fetch()` | `.entry('uid').fetch<T>()` |
| Entry query | `.Query().toJSON().find()` | `.entry().query().find<T>()` |
| Content type | `.ContentType('uid')` | `.contentType('uid')` |
| Locale | `.language('en-us')` | `.setLocale('en-us')` |
| References | `.includeReference(['ref'])` | `.includeReference('ref')` |
| Sync | `Stack.sync({ 'init': true })` | `stack.sync()` (no init param) |
| Type safety | None (manual casting) | Generic types: `fetch<T>()`, `find<T>()` |

**Guidance:**

- Method names changed from PascalCase to camelCase
- `.toJSON()` is no longer needed
- TypeScript generics replace manual type casting
- The Utils library (`@contentstack/utils`) is installed separately if needed for Rich Text rendering

## Output

Every response includes:

1. **Complete TypeScript code** in a fenced code block — copy-paste ready with all imports at the top
2. **Type definitions** — interfaces for the content model, always extending `BaseEntry` or `BaseAsset`
3. **Explanation** — a brief, clear description of what the code does and why specific methods were chosen
4. **Suggestions** (when applicable) — optional improvements like adding pagination, error handling, caching, or alternative approaches

**Format example:**

```
## Generated code

[TypeScript code block with imports, types, and implementation]

## What this does

- [Bullet point explanation of the query chain]
- [What data will be returned]

## Suggestions

- [Optional improvements or alternatives]
```

## Error handling and edge cases

- **Management operations**: If the user asks about creating, updating, or deleting entries/assets, explain that the Delivery SDK is read-only. Redirect to the Contentstack Management SDK or the `contentstack-cli-assistant` skill for write operations.
- **URL size limit**: The Content Delivery API has an 8KB URL size limit. If a query has many filters, large `containedIn` arrays, or extensive parameters, warn the user and suggest splitting into multiple queries.
- **Multiple content type references**: Querying across multiple content types in a single request is not supported. Suggest separate queries per content type.
- **Global Fields**: Global Field schemas cannot be queried directly. They are included as part of the content type schema when fetching content type details.
- **Node.js version**: The SDK requires Node.js 22+. If the user mentions compatibility issues, check their Node.js version first.
- **Region-specific hosts**: When the user operates outside AWS NA, ensure the correct region is configured. This affects both the API host and the Image Delivery API base URL.
- **Live Preview in server environments**: Always warn against sharing a single SDK instance across concurrent requests. Recommend creating a new instance per request.
- **Image transformation limits**: If the user requests transformations that exceed limits (50MB input, 8192x8192 output, 4096x4096 for AVIF), inform them of the constraints.
