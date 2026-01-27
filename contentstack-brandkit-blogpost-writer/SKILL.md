---
name: contentstack-brandkit-blogpost-writer
description: Fetch a Contentstack Brandkit voice profile and write a blogpost in that voice. Given a premise, generate clarifying questions, interview the user, then produce a polished blogpost plus a structured JSON draft matching the CMS content type schema.
compatibility: Requires Contentstack MCP Brandkit tools and CMA content type read access.
metadata:
  vendor: contentstack
  tools: brandkit, cma
  default_mode: write
---

## Goal

1. Load the correct Brandkit voice profile
2. Get the premise from the user
3. Generate clarifying questions based on the premise
4. Interview the user to gather answers
5. Discover the blog content type schema from Contentstack
6. Draft a blogpost using the voice profile, premise, and collected answers
7. Iterate until the post is good
8. Output:
   - final markdown blogpost
   - a structured JSON draft matching the CMS content type fields

## Inputs

Ask only for what is missing. Collect inputs conversationally across multiple steps:

### Initial inputs (ask first)

- voice_profile_selector:
  - preferred: voice_profile_uid
  - fallback: a voice profile name or "use my default"
- premise: one paragraph describing what the blogpost should cover

### Gathered during interview

- answers: collected from the clarifying questions you generate

### Content type discovery (ask after interview)

- content_type_uid: the Contentstack content type UID for blog posts
  - if unknown, list available content types and let user pick
- branch: string (default "main")

### Optional inputs

- audience: who the post is for
- desired_length: short | medium | long (default medium)
- include_sections: optional list (example: intro, main points, faq, conclusion)
- seo:
  - primary_keyword (optional)
  - meta_description (optional)
- revision_rounds: number (default 2)

If multiple voice profiles exist and the user did not specify one, list them and ask which to use.

## Tools

### Contentstack MCP Brandkit

- get_all_voice_profiles
- get_a_single_voice_profile (profile_id)

### Contentstack MCP CMA (content type discovery)

- get_all_content_types (branch, limit, skip, include_count)
- get_a_single_content_type (content_type_uid, branch, include_global_field_schema)

## Procedure

### 1) Collect premise and select voice profile

1. Ask the user for their **premise** — a paragraph describing what the blogpost should be about
2. Call `get_all_voice_profiles`
3. If `voice_profile_uid` provided, call `get_a_single_voice_profile(profile_id)`
4. Else if multiple profiles exist, list them and ask which to use
5. Once selected, call `get_a_single_voice_profile(profile_id)` to load full voice profile details

### 2) Generate clarifying questions

Based on the premise and voice profile context:

1. Generate 3-5 targeted questions that will help you write a better post
2. Questions should:
   - fill knowledge gaps about the topic
   - clarify the user's unique perspective or experience
   - gather specific examples, data, or anecdotes
   - understand the key takeaway the user wants readers to have
3. Present the questions to the user one at a time or as a numbered list
4. Wait for the user to answer before proceeding

### 3) Discover content type schema

Before writing, determine the JSON structure:

1. Ask the user for the `content_type_uid` for their blog posts
   - If unknown, call `get_all_content_types(branch, limit="100", skip="0", include_count=true)`
   - Filter for content types with names/uids containing: blog, post, article, news
   - Present matching options and ask user to pick one
2. Call `get_a_single_content_type(content_type_uid, branch, include_global_field_schema=true)`
3. Parse the schema to identify:
   - required fields
   - field UIDs and their types (text, markdown, rich text, etc.)
   - any reference fields or special structures
4. Store this schema for the JSON output step

### 4) Generate first draft

Use your own AI model to generate the first draft:

- Apply the voice profile tone, style, and guidelines
- Incorporate the premise and all collected answers
- Structure according to `include_sections` if provided
- Target the `desired_length`
- Consider the `audience` if specified
- While writing, identify potential FAQ questions that readers might have based on the article content

**Writing style requirements (always apply):**

- **Title capitalization**: All titles must use lowercase except the first word and acronyms (e.g., "Getting started with API integration" not "Getting Started With API Integration")
- **Dashes**: Never use em dash (—) or en dash (–). Use regular hyphens (-) or commas instead

### 5) Revision loop

For each revision round (default 2):

- Present the draft to the user
- Ask for targeted feedback or accept approval
- If revisions requested:
  - keep voice consistent
  - improve clarity, flow, and specificity
  - remove repetition
  - ensure the post answers the premise and uses the collected answers effectively
  - enforce writing style requirements: lowercase titles (except first word and acronyms), no em/en dashes

### 6) Output

Return two artifacts:

**A) Final blogpost in Markdown**

The complete, polished blogpost with front matter. Format:

```markdown
---
id: [auto-generated unique numeric ID]
slug:
  [
    automatically generated from the title in lowercase kebab-case; convert to lowercase,
    replace any non-alphanumeric character with a hyphen,
    collapse multiple hyphens,
    and trim leading/trailing hyphens,
  ]
title: [as written, with correct capitalization]
description: [TL;DR paragraph content]
date: [ISO timestamp, e.g., "2025-10-04T20:37:36Z"]
image: ""
canonical_url: [https://timbenniks.dev/writing/<slug>]
tags: [lowercase array of relevant taxonomy terms]
reading_time: [automatically estimated based on word count, e.g., "5 min read"]

head:
  meta:
    - property: twitter:image
      content: [same image URL]
    - property: twitter:title
      content: [same as title]
    - property: twitter:description
      content: [same as description]
    - property: keywords
      content: [comma-separated tags]

faqs:
  - question: [Q1]
    answer: [A1]
  - question: [Q2]
    answer: [A2]
  - question: [Q3]
    answer: [A3]
---

### Then the article body
```

**Front matter field requirements:**

- `id`: Generate a unique numeric ID (e.g., timestamp-based or sequential)
- `slug`: Auto-generate from title: lowercase, convert non-alphanumeric to hyphens, collapse multiple hyphens, trim edges
- `title`: Use the actual title as written (with correct capitalization per style rules)
- `description`: Extract or write a concise TL;DR paragraph summarizing the article
- `date`: Current ISO timestamp in format "YYYY-MM-DDTHH:MM:SSZ"
- `image`: Leave empty string "" (unless user provides one)
- `canonical_url`: Format as `https://timbenniks.dev/writing/<slug>` using the generated slug
- `tags`: Array of lowercase relevant taxonomy terms/keywords from the article
- `reading_time`: Estimate based on word count (e.g., "5 min read", "3 min read")
- `head.meta`: Twitter card meta tags and keywords using the same title, description, image, and tags
- `faqs`: Generate 3-5 FAQ items based on the article content. Questions should address common reader questions that arise from the article topics. Answers should be concise and reference the article content.

**Article body:** Follow the markdown front matter with the full article content.

**B) JSON draft matching CMS schema**

A JSON object with field UIDs from the discovered content type schema:

```json
{
  "<field_uid>": "<value>",
  "<title_field_uid>": "...",
  "<body_field_uid>": "...",
  ...
}
```

Mapping rules:

- Only include fields that exist in the content type schema
- Use the exact field UIDs from the schema
- For required fields: include them (ask user if content is missing)
- For markdown/rich text fields: use the appropriate format based on field type
- For optional fields: include if relevant content was gathered
- Omit fields with no corresponding content

**Note:** This skill does not create CMS entries. Use the `contentstack-cms-blogpost-entry-creator` skill to publish the JSON draft to Contentstack.
