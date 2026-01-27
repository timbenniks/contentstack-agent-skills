---
name: contentstack-cms-blogpost-entry-creator
description: Discover the correct Contentstack CMS content model for a blogpost, map the provided blog fields to the model schema, and create a new entry using Contentstack MCP (CMA). Use when you have blog content ready and want it inserted into the correct content type safely.
compatibility: Requires Contentstack MCP CMA tools with permission to read content types and create entries.
metadata:
  vendor: contentstack
  tools: cma
  default_mode: plan
---

## Goal

Given a blogpost payload (title, slug, body, etc):

1. Identify the correct content type
2. Map fields safely
3. Create a new entry in the right locale and branch
4. Verify the created entry
   Optional: publish only if the user explicitly asks

## Safety and scope

- Default mode is plan.
- Only mutate Contentstack in execute mode.
- Never publish unless explicitly asked (publish_an_entry is optional).
- Always show the final entry_data payload before creating the entry when in execute mode.

## Inputs

Ask only for what is missing:

- mode: plan | execute (default plan)
- branch: string (default "main")
- locale: string (default: use stack default locale if detectable, else ask)
- blog_payload:
  - title
  - body
- content_type_hint: optional string (example: "article")
- publish_after_create: true|false (default false)

## Tools (Contentstack MCP CMA)

Discovery:

- get_all_content_types (branch, limit, skip, include_count)
- get_a_single_content_type (content_type_uid, branch, include_global_field_schema=true)
- get_all_languages (limit, skip)

Create and verify:

- create_an_entry (branch, content_type_uid, locale, entry_data)
- get_single_entry (content_type_uid, entry_uid, locale, branch)
  Optional:
- update_an_entry (if a second pass is needed)
- publish_an_entry (only if explicitly requested)

## Procedure

### 1) Determine locale

1. Call get_all_languages with limit="100", skip="0", paginate until complete.
2. If user provided locale, validate it exists.
3. If not provided:
   - prefer the stack default locale if present in the response
   - otherwise ask the user to pick a locale

### 2) Identify best content type

1. Call get_all_content_types(branch="main" or selected branch) with:

   - include_count=true
   - limit="100"
   - skip="0"
     Paginate until complete.

2. Score content types:

   - strong match if uid or title contains: blog, post, article
   - score higher if schema includes fields resembling:
     - title (text)
     - body/content (markdown)

3. For the top candidates (up to 5), call get_a_single_content_type with:

   - content_type_uid
   - branch
   - include_global_field_schema=true
     Then rescore using full schema.

4. Select the best content_type_uid.
   If ambiguous, present the top 2 and ask which one to use.

### 3) Build entry_data mapping

Construct entry_data as:
{
"entry": {
"<field_uid>": <value>,
...
}
}

Mapping rules:

- Only map fields that exist in the selected schema.
- Respect required fields:
  - if required fields are missing from blog_payload, ask for them.
- For rich text or modular blocks:
  - if schema expects rich text, convert markdown to the expected format only if you have a deterministic mapping
  - otherwise put markdown into a plain text field or ask which field should receive it

### 4) Plan output (always)

Before executing:

- show selected content_type_uid
- show locale and branch
- show the final entry_data JSON that will be sent to create_an_entry

If mode=plan, stop here.

### 5) Create entry (execute only)

Call create_an_entry with:

- branch
- content_type_uid
- locale
- entry_data (object)

Capture the returned entry uid.

### 6) Verify entry

Call get_single_entry with:

- branch
- content_type_uid
- entry_uid
- locale

Confirm:

- title and slug match
- body field populated
- no validation errors indicated

### 7) Optional publish

Only if user explicitly requested publish_after_create=true:

- call publish_an_entry with the required publish params for your stack setup (environment, locales, etc)
  If publish requires extra inputs (environment), ask for them.

## Output

Return:

- created entry uid
- chosen content type uid
- locale and branch
- verification summary
- if published: publish result summary
