---
name: contentstack-js-to-ts-migration
description: Hands-on migration assistant for converting Contentstack JavaScript Delivery SDK code to the TypeScript Delivery SDK. Analyzes existing code, identifies changes, and generates migrated TypeScript with types.
compatibility: Requires Node.js 22+. Migrates from contentstack to @contentstack/delivery-sdk.
metadata:
  vendor: contentstack
  tools: code-generation
  default_mode: plan
---

## Goal

Act as a hands-on migration assistant that converts Contentstack JavaScript Browser Delivery SDK code (`contentstack`) to the TypeScript Delivery SDK (`@contentstack/delivery-sdk`). Given existing JavaScript code or a description of current usage:

1. Analyze the code to identify every SDK pattern that needs to change
2. Build a tailored migration checklist based on the specific features used
3. Generate complete, type-safe TypeScript replacements with proper imports and interfaces
4. Highlight breaking changes, removed APIs, and new features worth adopting
5. Present all changes for review before the user applies them

## Safety and scope

- Default mode is **plan**. Present migrated code for review. Do not write files unless the user explicitly asks to.
- **Never embed real credentials in generated code.** Use descriptive placeholder strings like `"your_api_key"`, `"your_delivery_token"`, `"your_environment"`. If the user provides real credentials, use them but remind them not to commit secrets to version control.
- **This skill covers Delivery SDK migration only.** If the user's code includes Management SDK operations (creating, updating, or deleting content), explain that those require the Contentstack Management SDK and are outside this skill's scope.
- **Always generate TypeScript with types.** Every migrated code snippet must include interfaces extending `BaseEntry` or `BaseAsset` and use generics (`fetch<T>()`, `find<T>()`).
- **Always include imports.** Every migrated snippet starts with the necessary import statements.
- **Preserve the user's business logic.** Only change SDK-specific patterns. Do not refactor unrelated application code unless asked.

## Inputs

Ask only for what is missing. Collect inputs conversationally.

### Migration context (gather at the start)

1. **Source code**: The user's existing JavaScript SDK code. Ask them to paste it or describe which SDK features they use.
2. **Content types**: Names and field structures of their content types (needed for generating TypeScript interfaces). If not provided, use generic placeholder types and note that the user should replace them.
3. **Region** (optional): Their Contentstack region (US, EU, AU, Azure NA, Azure EU, GCP NA, GCP EU). Defaults to US if not specified.
4. **Framework** (optional): What framework they use (Next.js, Gatsby, Express, etc.). Affects live preview and SSR migration advice.

### Per-request context

When the user asks to migrate a specific piece of code, gather:

- The JavaScript code snippet to migrate
- Any context about what the code does (if not obvious from the code itself)

## Tools to use

This skill generates code. No bash commands are needed. All output is TypeScript code presented in fenced code blocks for the user to copy into their project.

## Procedure

### 0) Assess current JS SDK usage

When the user shares their JavaScript code or describes their SDK usage, scan for these patterns and build a migration checklist:

| Pattern detected | Migration area | Severity |
|---|---|---|
| `Contentstack.Stack(...)` or `new Contentstack(...)` | Initialization (step 2) | Breaking |
| `import Contentstack from 'contentstack'` or `require('contentstack')` | Package & imports (step 1) | Breaking |
| `.ContentType(...)` / `.Entry(...)` / `.Query()` | Method names (step 3) | Breaking |
| `.toJSON()` | Removed API (step 12) | Breaking — must remove |
| `.where('field', value)` (2-arg) | Query operators (step 4) | Breaking — signature changed |
| `.notEqualTo(...)` / `.lessThan(...)` / `.greaterThan(...)` etc. | Query operators (step 4) | Breaking — replaced by `.where()` |
| `.language(...)` | Renamed method (step 5) | Breaking |
| `.ascending(...)` / `.descending(...)` | Renamed method (step 5) | Breaking |
| `.includeSchema()` | Removed API (step 12) | Breaking — use `.includeContentType()` |
| `.includeReferenceContentTypeUID()` | Removed API (step 12) | Breaking — removed |
| `.includeReference([...])` (array arg) | Changed signature (step 5) | Breaking — chain singles |
| `.Assets(...)` | Asset migration (step 7) | Breaking — renamed singular |
| `Stack.imageTransform(...)` | Image transform (step 8) | Breaking — full rewrite |
| `.setCachePolicy(...)` / `.getCacheProvider()` | Cache migration (step 9) | Breaking — config-based |
| `Stack.sync({init: true})` | Sync migration (step 10) | Breaking — param removed |
| `.then(function success(...), function error(...))` | Promise patterns (step 15) | Recommended |
| `Contentstack.Utils` | Utils extraction (step 14) | Breaking — separate package |
| `live_preview` config | Live preview (step 11) | Minor changes |
| `result[0]` / `result[result.length-1]` | Result structure (step 15) | Breaking |
| `.setHost(...)` / `.setPort(...)` / `.setProtocol(...)` | Init config (step 2) | Breaking — removed |
| `Stack.getLastActivities()` | Removed API (step 12) | Breaking — removed |
| `.count()` | Removed API (step 12) | Breaking — use `.includeCount()` |
| `.getQuery()` | Removed API (step 12) | Breaking — removed |

Present the checklist to the user:

```
## Migration checklist for your code

Based on your code, here are the changes needed:

- [ ] **Breaking**: Package & imports — change package and import statements
- [ ] **Breaking**: Initialization — rewrite Stack creation
- [ ] **Breaking**: Query operators — migrate .notEqualTo() etc. to .where()
- [ ] ...
- [ ] **Recommended**: Adopt async/await instead of .then() callbacks
- [ ] **Optional**: Add TypeScript interfaces for your content types

Shall I walk through each one, or migrate the entire file at once?
```

### 1) Package & import migration

**Before (JavaScript):**

```javascript
// npm package: contentstack
import Contentstack from 'contentstack';
// or
const Contentstack = require('contentstack');
```

**After (TypeScript):**

```typescript
// npm package: @contentstack/delivery-sdk
import contentstack from '@contentstack/delivery-sdk';

// Import specific types and enums as needed:
import contentstack, {
  Region,
  QueryOperation,
  BaseEntry,
  BaseAsset,
} from '@contentstack/delivery-sdk';

// For image transforms:
import {
  ImageTransform,
  FormatEnum,
  CropByEnum,
  FitEnum,
} from '@contentstack/delivery-sdk';
```

**What changed:**

- Package: `contentstack` → `@contentstack/delivery-sdk`
- Install: `npm uninstall contentstack && npm i @contentstack/delivery-sdk`
- Import: `Contentstack` (capital C) → `contentstack` (lowercase c)
- Enums and types are named exports, imported individually
- If the code uses `Contentstack.Utils`, also run: `npm i @contentstack/utils`

### 2) Stack initialization

**Before — positional arguments (JavaScript):**

```javascript
const Stack = Contentstack.Stack(
  "your_api_key",
  "your_delivery_token",
  "your_environment"
);
```

**Before — object configuration (JavaScript):**

```javascript
const Stack = Contentstack.Stack({
  'api_key': "your_api_key",
  'delivery_token': "your_delivery_token",
  'environment': "your_environment",
  'region': Contentstack.Region.EU
});
```

**After (TypeScript):**

```typescript
import contentstack, { Region } from '@contentstack/delivery-sdk';

const stack = contentstack.stack({
  apiKey: "your_api_key",
  deliveryToken: "your_delivery_token",
  environment: "your_environment",
  region: Region.EU,
});
```

**What changed:**

| Change | JavaScript | TypeScript |
|---|---|---|
| Factory method | `Contentstack.Stack()` (capital S) | `contentstack.stack()` (lowercase s) |
| Arguments | Positional strings or snake_case object | Named object with camelCase keys |
| API key param | `api_key` or positional arg 1 | `apiKey` |
| Token param | `delivery_token` or positional arg 2 | `deliveryToken` |
| Environment | `environment` or positional arg 3 | `environment` |
| Region | `Contentstack.Region.EU` | `Region.EU` (separate import) |
| Variable naming | `Stack` (capital) by convention | `stack` (lowercase) by convention |

**Host/port/protocol migration:**

```javascript
// Before (JavaScript) — setter methods:
Stack.setProtocol("https");
Stack.setHost("custom-cdn.example.com");
Stack.setPort(443);
```

```typescript
// After (TypeScript) — config parameter:
const stack = contentstack.stack({
  apiKey: "your_api_key",
  deliveryToken: "your_delivery_token",
  environment: "your_environment",
  host: "custom-cdn.example.com",
});
```

### 3) Content type & entry method names

All core access methods change from PascalCase to camelCase.

**Before (JavaScript):**

```javascript
// Single entry
const entry = await Stack.ContentType("blog_post")
  .Entry("blt1234567890abcdef")
  .toJSON()
  .fetch();

// Multiple entries
const result = await Stack.ContentType("blog_post")
  .Query()
  .toJSON()
  .find();

// Single entry shorthand
const result = await Stack.ContentType("blog_post")
  .Query()
  .toJSON()
  .findOne();
```

**After (TypeScript):**

```typescript
import contentstack, { BaseEntry } from '@contentstack/delivery-sdk';

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  author: string;
}

// Single entry
const entry = await stack
  .contentType("blog_post")
  .entry("blt1234567890abcdef")
  .fetch<BlogPost>();

// Multiple entries
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .find<BlogPost>();

// Single entry shorthand
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .findOne<BlogPost>();
```

**What changed:**

| JavaScript | TypeScript | Notes |
|---|---|---|
| `.ContentType('uid')` | `.contentType('uid')` | camelCase |
| `.Entry('uid')` | `.entry('uid')` | camelCase |
| `.Entry('uid').toJSON().fetch()` | `.entry('uid').fetch<T>()` | `.toJSON()` removed, generic added |
| `.Query()` | `.entry().query()` | Must call `.entry()` first |
| `.Query().toJSON().find()` | `.entry().query().find<T>()` | `.toJSON()` removed, generic added |
| `.findOne()` | `.findOne<T>()` | Generic added |

**Guidance:** The `.toJSON()` call is no longer needed because the TypeScript SDK always returns plain objects. Remove every `.toJSON()` call in query chains.

### 4) Query operator migration

This is the **largest breaking change**. The JavaScript SDK uses named comparison methods. The TypeScript SDK consolidates them into `.where()` with a `QueryOperation` enum.

**Before (JavaScript):**

```javascript
const result = await Stack.ContentType("product")
  .Query()
  .where("status", "published")
  .notEqualTo("category", "archived")
  .greaterThan("price", 50)
  .greaterThanOrEqualTo("rating", 4)
  .lessThan("stock", 100)
  .lessThanOrEqualTo("discount", 30)
  .toJSON()
  .find();
```

**After (TypeScript):**

```typescript
import contentstack, { QueryOperation, BaseEntry } from '@contentstack/delivery-sdk';

interface Product extends BaseEntry {
  status: string;
  category: string;
  price: number;
  rating: number;
  stock: number;
  discount: number;
}

const result = await stack
  .contentType("product")
  .entry()
  .query()
  .where("status", QueryOperation.EQUALS, "published")
  .where("category", QueryOperation.NOT_EQUALS, "archived")
  .where("price", QueryOperation.GREATER_THAN, 50)
  .where("rating", QueryOperation.GREATER_THAN_OR_EQUALS, 4)
  .where("stock", QueryOperation.LESS_THAN, 100)
  .where("discount", QueryOperation.LESS_THAN_OR_EQUALS, 30)
  .find<Product>();
```

**Complete operator mapping:**

| JavaScript method | TypeScript equivalent |
|---|---|
| `.where('field', value)` | `.where('field', QueryOperation.EQUALS, value)` |
| `.notEqualTo('field', value)` | `.where('field', QueryOperation.NOT_EQUALS, value)` |
| `.lessThan('field', value)` | `.where('field', QueryOperation.LESS_THAN, value)` |
| `.lessThanOrEqualTo('field', value)` | `.where('field', QueryOperation.LESS_THAN_OR_EQUALS, value)` |
| `.greaterThan('field', value)` | `.where('field', QueryOperation.GREATER_THAN, value)` |
| `.greaterThanOrEqualTo('field', value)` | `.where('field', QueryOperation.GREATER_THAN_OR_EQUALS, value)` |

**Methods that stay the same** (no change needed):

| Method | Notes |
|---|---|
| `.containedIn('field', [...])` | Same API |
| `.notContainedIn('field', [...])` | Same API |
| `.exists('field')` | Same API |
| `.notExists('field')` | Same API |
| `.regex('field', 'pattern', 'flags')` | Same API |
| `.tags([...])` | Same API |
| `.limit(n)` | Same API |
| `.skip(n)` | Same API |
| `.only([...])` | Same API |
| `.except([...])` | Same API |

**Logical operators:**

```javascript
// Before (JavaScript):
const query1 = Stack.ContentType("blog").Query().where("author", "John");
const query2 = Stack.ContentType("blog").Query().where("author", "Jane");
Stack.ContentType("blog").Query().or(query1, query2).toJSON().find();
```

```typescript
// After (TypeScript):
import { QueryOperatorEnum } from '@contentstack/delivery-sdk';

const query1 = stack.contentType("blog").entry().query()
  .where("author", QueryOperation.EQUALS, "John");
const query2 = stack.contentType("blog").entry().query()
  .where("author", QueryOperation.EQUALS, "Jane");

stack.contentType("blog").entry().query()
  .queryOperator(QueryOperatorEnum.OR, query1, query2)
  .find<BlogPost>();
```

### 5) Renamed methods

These methods exist in both SDKs but have different names.

| JavaScript | TypeScript | Context |
|---|---|---|
| `.language('en-us')` | `.locale('en-us')` | On entry queries |
| N/A | `.setLocale('en-us')` | On stack instance |
| `.ascending('field')` | `.orderByAscending('field')` | Sort order |
| `.descending('field')` | `.orderByDescending('field')` | Sort order |
| `.includeSchema()` | `.includeContentType()` | Different method — schema is removed |
| `.includeReferenceContentTypeUID()` | Removed — no equivalent | |
| `.includeReference(['ref1', 'ref2'])` | `.includeReference('ref1').includeReference('ref2')` | Array → chained single calls |

**Before (JavaScript):**

```javascript
const result = await Stack.ContentType("blog_post")
  .Query()
  .language("fr-fr")
  .ascending("published_date")
  .includeSchema()
  .includeReference(["author", "category"])
  .toJSON()
  .find();
```

**After (TypeScript):**

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .locale("fr-fr")
  .orderByAscending("published_date")
  .includeContentType()
  .includeReference("author")
  .includeReference("category")
  .find<BlogPost>();
```

### 6) New TypeScript-only features to adopt

After migrating, consider adopting these features that have no JavaScript equivalent:

| Feature | Method | What it does |
|---|---|---|
| Metadata | `.includeMetadata()` | Returns field-level metadata with entries/assets |
| Image dimensions | `.includeDimension()` | Returns width/height with asset responses |
| Full-text search | `.search('keyword')` | Search across all text fields in entries |
| OOP pagination | `.paginate().next()` / `.previous()` | Navigate pages without manual skip/limit math |
| Referenced field queries | `.whereIn('refField')` / `.whereNotIn('refField')` | Filter by referenced entry values |
| Multiple variants | `.variants(['uid1', 'uid2'])` | Fetch multiple personalization variants at once |
| Relative URLs | `.relativeUrls()` | Get relative asset URLs instead of absolute |
| Asset versioning | `.version(2)` | Fetch a specific version of an asset |
| Retry config | `fetchOptions: { retryLimit: 5, retryDelay: 300 }` | Built-in retry with backoff |
| Early access | `early_access: ['feature_name']` | Opt into preview features |
| Plugins | `plugins: [myPlugin]` | Extend SDK behavior |

**Example — adopting pagination:**

```javascript
// Before (JavaScript) — manual pagination:
let page = 0;
const pageSize = 20;
const result = await Stack.ContentType("blog")
  .Query()
  .skip(page * pageSize)
  .limit(pageSize)
  .toJSON()
  .find();
page++;
```

```typescript
// After (TypeScript) — OOP pagination:
const query = stack
  .contentType("blog")
  .entry()
  .query()
  .limit(20)
  .paginate();

const page1 = await query.find<BlogPost>();
const page2 = await query.next().find<BlogPost>();
const backToPage1 = await query.previous().find<BlogPost>();
```

### 7) Asset migration

**Before (JavaScript):**

```javascript
// Single asset
const asset = await Stack.Assets("blt1234567890abcdef")
  .toJSON()
  .fetch();

// All assets
const assets = await Stack.Assets()
  .Query()
  .toJSON()
  .find();
```

**After (TypeScript):**

```typescript
import contentstack, { BaseAsset } from '@contentstack/delivery-sdk';

interface ProjectAsset extends BaseAsset {
  title: string;
  description: string;
  url: string;
}

// Single asset
const asset = await stack
  .asset("blt1234567890abcdef")
  .fetch<ProjectAsset>();

// All assets
const assets = await stack
  .asset()
  .find<ProjectAsset>();
```

**What changed:**

| JavaScript | TypeScript | Notes |
|---|---|---|
| `Stack.Assets('uid')` | `stack.asset('uid')` | camelCase + **singular** |
| `Stack.Assets()` | `stack.asset()` | camelCase + **singular** |
| `.Assets().Query().toJSON().find()` | `.asset().find()` | No `.Query()` wrapper, no `.toJSON()` |
| `.toJSON().fetch()` | `.fetch<T>()` | `.toJSON()` removed, generic added |

### 8) Image transform rewrite

This is a **complete rewrite**. The JavaScript SDK uses a function that takes a URL and a params object. The TypeScript SDK uses an OOP chainable `ImageTransform` class.

**Before (JavaScript):**

```javascript
const imageURL = "https://images.contentstack.io/v3/assets/.../image.jpg";

// Resize
const resized = Stack.imageTransform(imageURL, {
  width: 400,
  height: 300
});

// Crop and convert
const transformed = Stack.imageTransform(imageURL, {
  crop: "200,300",
  format: "webp",
  quality: 80
});

// Multiple transforms
const optimized = Stack.imageTransform(imageURL, {
  width: 800,
  height: 600,
  format: "webp",
  quality: 85,
  auto: "webp"
});
```

**After (TypeScript):**

```typescript
import { ImageTransform, FormatEnum } from '@contentstack/delivery-sdk';

// Resize
const resized = new ImageTransform()
  .resize({ width: 400, height: 300 });

// Crop and convert
const transformed = new ImageTransform()
  .crop({ width: 200, height: 300 })
  .format(FormatEnum.WEBP)
  .quality(80);

// Multiple transforms
const optimized = new ImageTransform()
  .resize({ width: 800, height: 600 })
  .format(FormatEnum.WEBP)
  .quality(85)
  .auto();
```

**Parameter mapping:**

| JavaScript param object key | TypeScript ImageTransform method |
|---|---|
| `width`, `height` | `.resize({ width, height })` |
| `crop: "w,h"` | `.crop({ width, height })` |
| `crop: "w,h,x{x},y{y}"` | `.crop({ width, height, cropBy: CropByEnum.OFFSET, xval, yval })` |
| `format: "webp"` | `.format(FormatEnum.WEBP)` |
| `quality: 80` | `.quality(80)` |
| `auto: "webp"` | `.auto()` |
| `blur: 5` | `.blur(5)` |
| `sharpen: "a5,r2,t0"` | `.sharpen(5, 2, 0)` |
| `orient: 6` | `.orientation(6)` |
| `overlay: url` | `.overlay({ relativeUrl: url })` |
| `pad: "20,20,20,20"` | `.padding(20, 20, 20, 20)` |
| `trim: "10,10,10,10"` | `.trim(10, 10, 10, 10)` |
| `fit: "bounds"` | `.fit(FitEnum.BOUNDS)` |
| `dpr: 2` | `.dpr(2)` |

**Guidance:** The JavaScript SDK's `Stack.imageTransform()` returns a URL string directly. The TypeScript SDK's `ImageTransform` object is applied to assets through the SDK's asset handling. Review how your app consumes the transformed URL and update accordingly.

### 9) Cache policy migration

The caching system is **completely different** between SDKs.

**Before (JavaScript):**

```javascript
// Set cache policy on the Stack
Stack.setCachePolicy(Contentstack.CachePolicy.NETWORK_ELSE_CACHE);

// Set cache policy on a query
const query = Stack.ContentType("blog").Query();
query.setCachePolicy(Contentstack.CachePolicy.CACHE_THEN_NETWORK);

// Custom cache provider
Stack.setCacheProvider({
  get: async (key) => localStorage.getItem(key),
  set: async (key, value) => localStorage.setItem(key, value),
  getAll: async () => { /* ... */ }
});
```

**After (TypeScript):**

```typescript
import contentstack from '@contentstack/delivery-sdk';
// Also install: npm i @contentstack/persistance-plugin

const stack = contentstack.stack({
  apiKey: "your_api_key",
  deliveryToken: "your_delivery_token",
  environment: "your_environment",
  cacheOptions: {
    policy: "CACHE_THEN_NETWORK",
    storeType: "memoryStorage",
  },
});
```

**Cache policy mapping:**

| JavaScript (`Contentstack.CachePolicy.*`) | TypeScript (`cacheOptions.policy`) |
|---|---|
| `IGNORE_CACHE` | Omit `cacheOptions` entirely |
| `ONLY_NETWORK` | Omit `cacheOptions` (default behavior) |
| `CACHE_ELSE_NETWORK` | `"CACHE_THEN_NETWORK"` (closest equivalent) |
| `NETWORK_ELSE_CACHE` | `"CACHE_THEN_NETWORK"` (closest equivalent) |
| `CACHE_THEN_NETWORK` | `"CACHE_THEN_NETWORK"` |

**What changed:**

- No more `.setCachePolicy()` method on Stack or Query objects
- No more `.setCacheProvider()` / `.getCacheProvider()` methods
- Cache is configured once at initialization via `cacheOptions`
- Custom storage requires the `@contentstack/persistance-plugin` package
- Per-query cache policies are no longer supported — it's stack-level only

### 10) Sync API migration

**Before (JavaScript):**

```javascript
// Initial sync — note the {init: true} parameter
const initialResult = await Stack.sync({ 'init': true });

// Initial sync with filters
const filteredResult = await Stack.sync({
  'init': true,
  'locale': 'en-us',
  'content_type_uid': 'blog_post',
  'start_from': '2024-01-01'
});

// Pagination sync
const nextBatch = await Stack.sync({
  'pagination_token': initialResult.paginationToken
});

// Delta sync
const delta = await Stack.sync({
  'sync_token': initialResult.syncToken
});
```

**After (TypeScript):**

```typescript
// Initial sync — no {init: true} needed
const initialResult = await stack.sync();

// Initial sync with filters
const filteredResult = await stack.sync({
  locale: 'en-us',
  content_type_uid: 'blog_post',
  start_date: '2024-01-01',
});

// Pagination sync
const nextBatch = await stack.sync({
  pagination_token: initialResult.pagination_token,
});

// Delta sync
const delta = await stack.sync({
  sync_token: initialResult.sync_token,
});
```

**What changed:**

| JavaScript | TypeScript |
|---|---|
| `Stack.sync({init: true})` | `stack.sync()` — no init parameter |
| `Stack.sync({init: true, locale: '...'})` | `stack.sync({ locale: '...' })` |
| `start_from` parameter | `start_date` parameter |
| `result.paginationToken` (camelCase) | `result.pagination_token` (snake_case) |
| `result.syncToken` (camelCase) | `result.sync_token` (snake_case) |

**Complete sync flow (TypeScript):**

```typescript
// Initial full sync
let result = await stack.sync();

// Handle paginated results
while (result.pagination_token) {
  result = await stack.sync({
    pagination_token: result.pagination_token,
  });
}

// Store the sync token for later
const syncToken = result.sync_token;

// Later: fetch only changes since last sync
const delta = await stack.sync({ sync_token: syncToken });
```

### 11) Live preview migration

**Before (JavaScript):**

```javascript
const Stack = Contentstack.Stack({
  'api_key': "your_api_key",
  'delivery_token': "your_delivery_token",
  'environment': "your_environment",
  'live_preview': {
    'enable': true,
    'preview_token': "your_preview_token",
    'host': "rest-preview.contentstack.com"
  }
});

// SSR — extract live preview params from request
app.use((req, res, next) => {
  Stack.livePreviewQuery(req.query);
  next();
});
```

**After (TypeScript):**

```typescript
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

// SSR — same pattern, lowercase variable
app.use((req, res, next) => {
  stack.livePreviewQuery(req.query);
  next();
});
```

**What changed:**

- Config keys: `api_key` → `apiKey`, `delivery_token` → `deliveryToken` (the `live_preview` sub-object keys stay snake_case)
- Variable: `Stack` → `stack`
- Everything else stays the same

**Regional hosts:**

| Region | Preview host |
|---|---|
| AWS NA (default) | `rest-preview.contentstack.com` |
| AWS EU | `eu-rest-preview.contentstack.com` |
| AWS AU | `au-rest-preview.contentstack.com` |
| Azure NA | `azure-na-rest-preview.contentstack.com` |
| Azure EU | `azure-eu-rest-preview.contentstack.com` |
| GCP NA | `gcp-na-rest-preview.contentstack.com` |
| GCP EU | `gcp-eu-rest-preview.contentstack.com` |

**Guidance:** In server environments (Next.js, Express), do not share a single SDK instance across concurrent requests when Live Preview is enabled. Create a new stack instance per request to avoid cross-request data leakage.

### 12) Removed APIs — complete reference

These JavaScript SDK methods have been removed in the TypeScript SDK.

| Removed method | Replacement | Notes |
|---|---|---|
| `.toJSON()` | Remove it | TypeScript SDK always returns plain objects |
| `.includeSchema()` | `.includeContentType()` | Different method name and slightly different response shape |
| `.includeReferenceContentTypeUID()` | None | Removed with no replacement |
| `Stack.imageTransform(url, params)` | `new ImageTransform().method()...` | Complete rewrite to OOP class (see step 8) |
| `Stack.setCachePolicy(policy)` | `cacheOptions` in stack config | Set at initialization only (see step 9) |
| `Stack.setCacheProvider(provider)` | `@contentstack/persistance-plugin` | Separate package |
| `Stack.getCacheProvider()` | None | No equivalent |
| `Stack.setProtocol(proto)` | `host` config parameter | Include protocol in the host URL |
| `Stack.setHost(host)` | `host` config parameter | Set at initialization |
| `Stack.setPort(port)` | `host` config parameter | Include port in the host URL |
| `Stack.getLastActivities()` | None | Removed with no replacement |
| `.count()` | `.includeCount()` | Different approach — returns count alongside data |
| `.getQuery()` | None | Removed with no replacement |
| `{init: true}` sync parameter | Just call `stack.sync()` | Initial sync is the default behavior |

### 13) Type generation

The biggest advantage of the TypeScript SDK is type safety. Generate interfaces for every content type.

**Before (JavaScript) — no types:**

```javascript
const entry = await Stack.ContentType("blog_post")
  .Entry("blt1234567890abcdef")
  .toJSON()
  .fetch();

// No type checking — easy to make typos
console.log(entry.tilte); // typo goes unnoticed
```

**After (TypeScript) — full type safety:**

```typescript
import { BaseEntry, BaseAsset } from '@contentstack/delivery-sdk';

interface Author extends BaseEntry {
  name: string;
  bio: string;
  avatar: BaseAsset;
}

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  summary: string;
  author: Author;
  tags: Array<'tech' | 'design' | 'business'>;
  featured_image: BaseAsset;
  published_date: string;
  is_featured: boolean;
}

const entry = await stack
  .contentType("blog_post")
  .entry("blt1234567890abcdef")
  .fetch<BlogPost>();

// Type checking catches typos at compile time
console.log(entry.tilte); // TS Error: Property 'tilte' does not exist
console.log(entry.title); // Works correctly
```

**Contentstack field → TypeScript type mapping:**

| Contentstack field type | TypeScript type |
|---|---|
| Single-line text | `string` |
| Multi-line text | `string` |
| Rich Text Editor (HTML) | `string` |
| Rich Text Editor (JSON) | `object` |
| Markdown | `string` |
| Number | `number` |
| Boolean | `boolean` |
| Date | `string` (ISO 8601 format) |
| File (single) | `BaseAsset` |
| File (multiple) | `BaseAsset[]` |
| Reference (single) | Custom interface extending `BaseEntry` |
| Reference (multiple) | Array of custom interface |
| Group | Inline object type `{ field: type; }` |
| Modular Blocks | Union type of block interfaces |
| Select (single) | String literal union: `'opt1' \| 'opt2'` |
| Select (multiple) | `Array<'opt1' \| 'opt2'>` |
| Link | `{ title: string; href: string }` |
| JSON | `Record<string, unknown>` |
| Taxonomy | `{ taxonomy_uid: string; term_uid: string }` |

**Guidance:** Ask the user to describe their content types or paste their content type JSON schema. Generate matching TypeScript interfaces for each content type and its referenced types.

### 14) Utils library extraction

**Before (JavaScript) — built-in:**

```javascript
import Contentstack from 'contentstack';

// Utils available directly on the Contentstack object
Contentstack.Utils.render({
  entry: entry,
  renderOption: {
    // render options
  }
});

// Or for JSON RTE
Contentstack.Utils.jsonToHTML({
  entry: entry,
  paths: ['body'],
  renderOption: {
    // render options
  }
});
```

**After (TypeScript) — separate package:**

```typescript
// Install separately: npm i @contentstack/utils
import { jsonToHTML, render } from '@contentstack/utils';

// Same API, different import
render({
  entry: entry,
  renderOption: {
    // render options
  }
});

jsonToHTML({
  entry: entry,
  paths: ['body'],
  renderOption: {
    // render options
  }
});
```

**What changed:**

- Install: `npm i @contentstack/utils` (separate package, not bundled)
- Import: `Contentstack.Utils.methodName` → `import { methodName } from '@contentstack/utils'`
- The utility functions themselves have the same API — only the import path changes

### 15) Promise pattern migration

**Before (JavaScript) — callback-style promises:**

```javascript
Stack.ContentType("blog_post")
  .Query()
  .where("status", "published")
  .ascending("published_date")
  .limit(10)
  .includeSchema()
  .includeCount()
  .toJSON()
  .find()
  .then(
    function success(result) {
      // JS SDK returns a mixed array:
      var entries = result[0];              // Entry objects
      var schema = result[1];              // Schema (if includeSchema)
      var count = result[result.length - 1]; // Count (if includeCount)
      entries.forEach(function(entry) {
        console.log(entry.title);
      });
    },
    function error(err) {
      console.error(err);
    }
  );
```

**After (TypeScript) — async/await with types:**

```typescript
import contentstack, { QueryOperation, BaseEntry } from '@contentstack/delivery-sdk';

interface BlogPost extends BaseEntry {
  title: string;
  body: string;
  status: string;
  published_date: string;
}

try {
  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .where("status", QueryOperation.EQUALS, "published")
    .orderByAscending("published_date")
    .limit(10)
    .includeContentType()
    .includeCount()
    .find<BlogPost>();

  // TypeScript SDK returns structured, typed response
  // Access entries and count directly from the typed result
  console.log(result);
} catch (err) {
  console.error(err);
}
```

**What changed:**

| JavaScript pattern | TypeScript pattern |
|---|---|
| `.then(success, error)` | `try { await ... } catch (err)` |
| `result[0]` for entries | Typed response object |
| `result[result.length - 1]` for count | Count included in typed response |
| `result[1]` for schema | Not applicable — use `.includeContentType()` |
| `function success(result)` | `const result = await ...` |
| No type checking on result | Full type inference from `<T>` generic |

### 16) Complete migration walkthrough

When the user wants to migrate their entire codebase, walk through these steps in order:

```
## Full migration checklist

### Phase 1: Setup
- [ ] 1. Install new package: `npm uninstall contentstack && npm i @contentstack/delivery-sdk`
- [ ] 2. Install utils (if used): `npm i @contentstack/utils`
- [ ] 3. Install cache plugin (if caching used): `npm i @contentstack/persistance-plugin`

### Phase 2: Foundation
- [ ] 4. Update all import statements (step 1)
- [ ] 5. Rewrite Stack initialization (step 2)
- [ ] 6. Generate TypeScript interfaces for all content types (step 13)

### Phase 3: Core queries
- [ ] 7. Update ContentType/Entry/Query method names — PascalCase to camelCase (step 3)
- [ ] 8. Remove all .toJSON() calls (step 12)
- [ ] 9. Migrate query operators — .notEqualTo() etc. to .where() (step 4)
- [ ] 10. Update renamed methods — .language(), .ascending(), etc. (step 5)

### Phase 4: Features
- [ ] 11. Migrate asset queries — .Assets() to .asset() (step 7)
- [ ] 12. Rewrite image transforms — function to ImageTransform class (step 8)
- [ ] 13. Migrate cache configuration (step 9)
- [ ] 14. Update sync calls — remove {init: true} (step 10)
- [ ] 15. Update live preview config (step 11)
- [ ] 16. Extract Utils to separate package (step 14)

### Phase 5: Modernize
- [ ] 17. Convert .then() callbacks to async/await (step 15)
- [ ] 18. Add generic types to all .fetch<T>() and .find<T>() calls
- [ ] 19. Remove references to any removed APIs (step 12)
- [ ] 20. Consider adopting new TS-only features (step 6)

### Phase 6: Verify
- [ ] 21. Run TypeScript compiler — fix any type errors
- [ ] 22. Test all SDK calls against your development environment
- [ ] 23. Verify live preview still works (if applicable)
- [ ] 24. Verify sync flow still works (if applicable)
```

Present this checklist to the user and offer to walk through each phase, or migrate specific files one at a time.

## Output

Every migration response includes:

1. **Before** — the user's JavaScript code (or a representative JS pattern)
2. **After** — the migrated TypeScript with imports, interfaces, and proper method chains
3. **What changed** — bullet list of every specific change made and why
4. **New opportunities** — TypeScript-only features the user can now adopt to improve their code

**Format:**

```
## Before (JavaScript)

[JS code block]

## After (TypeScript)

[TS code block with imports and interfaces]

## What changed

- [Change 1]: [reason]
- [Change 2]: [reason]
- ...

## New opportunities

- [Feature]: [brief description of what it enables]
```

## Error handling and edge cases

- **Management SDK code detected**: If the user's code includes content creation, updates, or deletes, explain that this skill covers the Delivery SDK (read-only) migration only. The Management SDK is a separate package with a different migration path.
- **Server-side Node.js SDK**: If the user is using `contentstack` in a Node.js server context (not browser), the migration path is the same — the TypeScript SDK works in both Node.js and browser environments.
- **Mixed SDK usage**: If the user uses both Delivery and Management SDKs, migrate the Delivery SDK calls first and leave the Management SDK calls unchanged. Flag them for a separate migration effort.
- **Deprecated JS patterns**: If the code uses very old patterns like `Stack.setProtocol()` or `Stack.getLastActivities()`, note that these have no TypeScript equivalent and suggest modern alternatives.
- **Cache complexity**: If the user has complex per-query cache policies in JavaScript, explain that TypeScript only supports stack-level caching and help them design an approach that works within that constraint.
- **Large codebases**: For large migrations, recommend proceeding file by file rather than all at once. Offer to prioritize the most-used SDK patterns first.
- **URL size limit**: Both SDKs share the 8KB URL size limit. If the user's queries are already close to this limit, the migration won't change that constraint.
