---
name: contentstack-release-runbook
description: Create, clone, populate, and deploy Contentstack releases using the Contentstack MCP (CMA). Use for launch coordination, scheduled publishing, release verification, and publish queue triage.
compatibility: Requires Contentstack MCP CMA tools. Needs permissions to create releases, add items, and deploy.
metadata:
  vendor: contentstack
  tools: cma
  default_mode: plan
---

## Goal

Produce a reliable release workflow that can:

- plan a release (dry run with validation)
- create or clone the release
- add items to the release
- verify release contents
- deploy now or schedule deployment
- verify and triage publish queue after deploy

## Safety and scope

- Default mode is **plan**. Do not write anything unless the user explicitly asks you to execute.
- Only call write tools in execute mode:
  - create_a_release, clone_a_release, add_items_to_a_release, delete_items_from_a_release, deploy_a_release
- Never infer “go live”. If the user asks for a plan, stay in plan mode.
- Always operate in the user-specified branch (or default to main).

## Inputs

Ask only for what is missing:

- mode: plan | execute (default plan)
- branch: string (default "main")
- release strategy: create | clone
- release_name: string
- release_description: string (optional)
- clone_release_id: string (required if strategy=clone)
- items_to_add: list of items to publish/unpublish in the release
- environments: list of environment names for deploy
- scheduled_at: optional ISO timestamp string for scheduled deploy (use the format expected by deploy_a_release)

### Item shape

Represent items internally as objects like:

- uid: string
- version: number
- locale: string (example "en-us")
- content_type_uid: string (for assets use "built_io_upload")
- action: string ("publish" or "unpublish")

The MCP tool add_items_to_a_release expects `item_data` as a JSON string containing `{ "items": [...] }`.

## Tools to use (Contentstack MCP CMA)

Read:

- get_all_releases (limit/skip pagination)
- get_a_single_release
- get_all_items_in_a_release
- get_publish_queue (limit/skip pagination)
- get_single_entry (optional for verification)
- get_a_single_asset (optional for verification)

Write:

- create_a_release
- clone_a_release
- add_items_to_a_release
- delete_items_from_a_release
- deploy_a_release

## Procedure

### 1) Resolve branch and baseline context

- Use branch = provided value or "main".
- If you need to list candidate releases, call:
  - get_all_releases with:
    - branch
    - include_count=true
    - include_items_count=true
    - limit="100"
    - skip="0"
  - paginate by increasing skip by limit until no more results.

### 2) Plan phase (always do this, even in execute)

1. Validate items_to_add:
   - ensure each item has uid, version, locale, content_type_uid, action
   - for assets ensure content_type_uid is "built_io_upload"
2. If possible, spot check a small sample:
   - for entries: get_single_entry (uid, content_type_uid, locale)
   - for assets: get_a_single_asset (uid)
3. Prepare the exact `item_data` JSON string:
   - { "items": [ ... ] }
4. Produce a “plan output” section:
   - branch
   - release strategy
   - release name
   - number of items to add, grouped by action and content_type_uid
   - environments and scheduled_at (if provided)

If mode=plan, stop here and output the plan.

### 3) Create or clone the release (execute only)

If strategy=create:

- call create_a_release with:
  - name
  - description (if provided)
  - include_branch=true

If strategy=clone:

- call clone_a_release with:
  - release_id (clone_release_id)
  - name (release_name)
  - description (optional)
  - include_branch=true

Capture the resulting release_id.

### 4) Add items to the release (execute only)

- call add_items_to_a_release with:
  - branch
  - release_id
  - item_data (stringified JSON)
  - include_branch=true

### 5) Verify release contents (always)

- call get_all_items_in_a_release with:
  - branch
  - release_id
  - include_branch=true

Verify:

- item count matches expectation
- actions match expectation
- locales look correct

If something is wrong:

- output a correction plan
- optionally (execute only) call delete_items_from_a_release to remove incorrect items, then re-add.

### 6) Deploy the release (execute only)

- call deploy_a_release with:
  - release_id
  - environments (array of environment names)
  - scheduled_at (omit if deploying immediately, include if scheduling)

### 7) Post-deploy verification and triage

1. Check publish queue:
   - call get_publish_queue with:
     - branch
     - include_count=true
     - limit="100"
     - skip="0"
   - paginate until done
2. Summarize:
   - failures and common error patterns
   - impacted environments, locales, content types
3. For top failing items, fetch details:
   - entries: get_single_entry
   - assets: get_a_single_asset

## Output

Return:

1. A concise markdown runbook report
2. A structured JSON summary at the end (for agents), with:
   - branch, release_id, release_name
   - items_added_count, deploy_environments, scheduled_at
   - publish_queue_summary (counts by status if available)
   - notes and next_actions
