---
name: contentstack-stack-inventory
description: Generate a complete inventory report for a Contentstack stack using the Contentstack MCP. Includes content types and schemas, global fields, languages/locales, branches and branch aliases, environments, taxonomies and term structure. Use when auditing a stack, onboarding to an existing implementation, preparing migrations, or documenting the current information architecture.
compatibility: Requires a host agent that can call the installed Contentstack MCP tools (CMA group) with access to the target stack credentials.
metadata:
  vendor: contentstack
  product_area: cma
  output: markdown
---

## Goal

Produce a single Markdown report that captures the stack's “information architecture” and key configuration:

- Data models (content types) with schema details
- Global fields and their schemas
- Languages/locales
- Branches and branch aliases
- Environments
- Taxonomies and term structure (plus export links or exported payload summaries)

## Inputs to ask for (only if missing)

- Branch to inspect (default: `main`)
- Whether to include taxonomy term deep details (default: yes, but summarize if very large)

Do not ask for stack credentials here; assume the Contentstack MCP is already authenticated and scoped to the intended stack.

## Tools to use (Contentstack MCP CMA)

Models:

- get_all_content_types (paginate with limit/skip)
- get_a_single_content_type (for full schema per content type)

Global fields:

- get_all_global_fields
- get_a_single_global_field

Languages:

- get_all_languages (paginate if needed)

Branches:

- get_all_branches (paginate)
- get_all_branch_aliases (paginate)

Environments:

- get_all_environments

Taxonomy:

- get_all_taxonomies (paginate)
- get_a_single_taxonomy (per taxonomy)
- get_all_terms (per taxonomy, paginate)
- get_a_single_term (spot checks or when you need extra metadata)
- get_all_descendants_of_a_term / get_all_ancestors_of_a_term (only when needed to explain structure)
- export_a_taxonomy (prefer JSON export)

## Procedure

### 1) Discover stack-wide configuration

1. Fetch languages:
   - Call get_all_languages repeatedly using limit=100 and skip until exhausted.
2. Fetch branches:
   - Call get_all_branches repeatedly using limit=100 and skip until exhausted.
3. Fetch branch aliases:
   - Call get_all_branch_aliases repeatedly using limit=100 and skip until exhausted.
4. Fetch environments:
   - Call get_all_environments with include_count=false (or true if you want the count).

### 2) Inventory models and global fields

1. Content types:
   - Call get_all_content_types for the chosen branch with include_global_field_schema=true.
   - Paginate using limit/skip.
   - For each content type UID, call get_a_single_content_type with include_global_field_schema=true to capture the canonical schema.
2. Global fields:
   - Call get_all_global_fields for the chosen branch.
   - For each global field UID, call get_a_single_global_field to capture its schema.

### 3) Inventory taxonomies

1. Taxonomy list:
   - Call get_all_taxonomies repeatedly using limit=100 and skip until exhausted.
2. For each taxonomy:
   - Call get_a_single_taxonomy to capture settings and counts.
   - Call export_a_taxonomy (format=json) to capture structure. If the export is very large, summarize the export (root terms count, max depth, example paths).
   - Call get_all_terms (taxonomy_uid) with include_children_count=true and include_referenced_entries_count=true where available. Paginate with limit/skip.
   - Compute simple metrics for the report:
     - total terms
     - approximate max depth (if export provides depth, use it; else infer from parent relationships if present)
     - top 10 terms by referenced entries count (if provided)

### 4) Produce the report

Output a Markdown document with these sections:

- Summary
  - branch inspected
  - counts: content types, global fields, languages, branches, environments, taxonomies, terms
- Languages
  - list of locales, default locale if present
- Branches
  - branches list
  - branch aliases list and what they point to (if present)
- Environments
  - each environment name and base URLs (if present)
- Content types
  - For each content type:
    - uid, title, description
    - options (localization enabled, etc, if present)
    - fields overview:
      - field uid, display name, data type
      - references (content types), modular blocks, group/global field usage
- Global fields
  - For each global field:
    - uid, title
    - fields overview
- Taxonomies
  - For each taxonomy:
    - uid, name, settings
    - counts
    - term structure summary (max depth, root term count)
    - notable terms (top referenced)
    - mention that a JSON export was pulled (include a short excerpt only)

## Output requirements

- Be deterministic and complete.
- Handle pagination everywhere (limit/skip).
- If any tool call fails, continue with what you have and note missing sections explicitly.
- Keep the main report human readable, but include a final “Raw data references” section listing which tool calls were used and on what UIDs.
