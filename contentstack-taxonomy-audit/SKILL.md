---
name: contentstack-taxonomy-audit
description: Audit Contentstack taxonomies and terms using the Contentstack MCP (CMA). Exports taxonomy JSON, computes health metrics (depth, orphans, heavy terms), and produces an action-oriented report.
compatibility: Requires Contentstack MCP CMA tools. Read-only by default.
metadata:
  vendor: contentstack
  tools: cma
  default_mode: audit
---

## Goal

Generate a taxonomy health report that includes:

- taxonomy inventory with counts
- export summaries (JSON export per taxonomy)
- term-level metrics:
  - root term count
  - depth estimate
  - orphan detection (best effort)
  - top referenced terms (if counts available)
  - naming and ordering anomalies (best effort)

## Safety and scope

- Default is read-only. Do not call update_a_taxonomy or update_a_term unless the user explicitly requests changes.
- If term volume is high, prefer:
  - export_a_taxonomy + summary metrics
  - plus get_all_terms with pagination but limit deep per-term fetches

## Inputs

Ask only for what is missing:

- branch: string (default "main") if your MCP setup requires it elsewhere (taxonomy tools here do not take branch)
- include_terms: true | false (default true)
- export_format: "json" (default json)
- max_terms_to_analyze: number (default 5000)
- deep_term_inspection:
  - none (default)
  - top_referenced (inspect top 20)
  - sample (inspect random N)

## Tools to use (Contentstack MCP CMA)

Taxonomies:

- get_all_taxonomies
- get_a_single_taxonomy
- export_a_taxonomy

Terms:

- get_all_terms
- get_a_single_term
- get_all_ancestors_of_a_term
- get_all_descendants_of_a_term
- get_all_terms_across_all_taxonomies (only for cross-taxonomy search)

Write (only if explicitly requested):

- update_a_taxonomy
- update_a_term

## Procedure

### 1) Inventory taxonomies

- call get_all_taxonomies with:
  - include_count=true
  - include_terms_count=true
  - include_referenced_entries_count=true
  - include_referenced_terms_count=true
  - limit="100"
  - skip="0"
- paginate by increasing skip by limit until exhausted.
- store the taxonomy_uids.

For each taxonomy_uid:

- call get_a_single_taxonomy with:
  - taxonomy_uid
  - include_terms_count=true
  - include_referenced_entries_count=true
  - include_referenced_terms_count=true

### 2) Export taxonomies

For each taxonomy_uid:

- call export_a_taxonomy with:
  - taxonomy_uid
  - format="json"

If the export payload is huge:

- do not paste the full export in the report
- extract summary stats only:
  - number of root terms
  - estimated depth (if the export includes hierarchy)
  - example paths (up to 5)

### 3) Term analysis (optional, bounded)

If include_terms=true:

For each taxonomy_uid:

- call get_all_terms with:
  - taxonomy_uid
  - include_children_count=true
  - include_referenced_entries_count=true
  - include_order=true
  - include_count=true
  - limit="100"
  - skip="0"
  - depth: omit (root-based default) unless the user requests a specific depth window

Paginate until:

- no more results, or
- analyzed terms reaches max_terms_to_analyze (then stop and mark as truncated)

Compute metrics per taxonomy:

- total terms analyzed
- root term estimate:
  - prefer export data if it includes explicit roots
  - else approximate by terms that have no parent info (if available) or by depth=0 in export
- “heavy” terms:
  - top 10 by referenced entries count (if present)
- “wide” nodes:
  - top 10 by children_count (if present)
- ordering anomalies:
  - duplicate order values among siblings (best effort if parent info exists)
- naming checks:
  - empty names, suspicious duplicates, inconsistent casing (best effort)

### 4) Deep inspection (optional)

If deep_term_inspection is enabled and the tools allow:

- For selected terms, call get_a_single_term with:
  - taxonomy_uid
  - term_uid
  - include_children_count=true
  - include_referenced_entries_count=true

For explaining structure:

- call get_all_ancestors_of_a_term for a problematic term
- call get_all_descendants_of_a_term for a heavy or wide term (use depth to bound)

### 5) Output

Produce a markdown report with:

- Summary
  - totals: taxonomies, terms analyzed, truncated flags
- Taxonomy list
  - uid, name, counts, notes
- Per taxonomy section
  - settings overview (from get_a_single_taxonomy)
  - export summary
  - metrics table: roots, depth estimate, heavy terms, wide terms
  - issues and recommendations

Append a machine-friendly JSON block:

- taxonomies[]
- metrics_by_taxonomy{}
- notable_terms[]
- truncated: true|false
- missing_capabilities[] (if any tool data was unavailable)

## Optional remediation (only if requested)

If the user asks to rename:

- call update_a_taxonomy with taxonomy_uid, name, description
- call update_a_term with taxonomy_uid, term_uid, name

Always output a pre-change plan and list of intended changes before executing updates.
