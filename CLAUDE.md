# LLM Wiki — Schema

You are maintaining a personal knowledge wiki. The wiki is a persistent, compounding artifact — not a chatbot conversation. Your job is to write, update, and maintain the wiki so the human can focus on sourcing, exploring, and thinking.

## Directory structure

```
raw/              # Source documents (immutable — never modify)
raw/assets/       # Images and attachments from sources
wiki/             # LLM-generated markdown pages (you own this entirely)
wiki/index.md     # Content catalog — update on every ingest
wiki/log.md       # Chronological activity log — append on every operation
CLAUDE.md         # This file — schema and conventions
```

## Page types

### Source summaries (`wiki/sources/`)
One page per ingested source. Filename: `source-YYYY-MM-DD-slug.md`

Frontmatter:
```yaml
---
title: "Original title"
source_type: article | paper | book_chapter | podcast | video | note | data
author: "Author name"
date: YYYY-MM-DD        # Publication date (if known)
ingested: YYYY-MM-DD    # When you processed it
file: "raw/filename"    # Path to raw source
tags: [tag1, tag2]
---
```

Body: 3-5 paragraph summary of key claims, data, and arguments. End with a "Connections" section listing links to entity/concept pages that were created or updated.

### Entity pages (`wiki/entities/`)
Pages for people, organizations, products, places — anything with a proper name that appears across multiple sources. Filename: `entity-slug.md`

Frontmatter:
```yaml
---
title: "Entity Name"
type: person | organization | product | place | project
first_seen: "source-slug"
source_count: 3
tags: [tag1, tag2]
---
```

Body: Running description synthesized from all sources. Include a "Sources" section at the bottom listing which source pages mention this entity.

### Concept pages (`wiki/concepts/`)
Pages for ideas, frameworks, theories, recurring themes. Filename: `concept-slug.md`

Frontmatter:
```yaml
---
title: "Concept Name"
first_seen: "source-slug"
source_count: 2
tags: [tag1, tag2]
---
```

Body: Explanation of the concept, how different sources treat it, tensions or contradictions between views. Link to related concepts and entities.

### Analysis pages (`wiki/analysis/`)
Pages generated from queries, comparisons, or synthesis. These are the "filed answers" — valuable outputs that shouldn't disappear into chat history. Filename: `analysis-slug.md`

Frontmatter:
```yaml
---
title: "Analysis Title"
created: YYYY-MM-DD
query: "The question that prompted this"
sources_used: [source-slug-1, source-slug-2]
tags: [tag1, tag2]
---
```

### Personal pages (`wiki/personal/`)
For the personal knowledge use case: goals, reflections, self-observations, patterns, health tracking, decisions. Filename: `personal-slug.md`

Frontmatter:
```yaml
---
title: "Page Title"
type: goal | reflection | pattern | decision | health | journal
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
---
```

### Overview (`wiki/overview.md`)
A single page that provides a high-level narrative of the entire wiki's contents. What does this knowledge base know? What are the major themes? What's well-covered and what has gaps? Update this after every few ingests or when the human asks.

## Cross-referencing conventions

- Use Obsidian-style wiki links: `[[page-name]]` or `[[page-name|Display Text]]`
- Every page should link to at least 2 other pages. Orphans are a lint failure.
- When a concept or entity is mentioned on any page, link it if a page exists for it.
- When creating a new entity/concept page, grep the existing wiki for mentions and add links retroactively.

## Workflows

### Ingest a source

1. Read the source document from `raw/`.
2. Discuss key takeaways with the human. Ask: anything to emphasize or skip?
3. Create a source summary page in `wiki/sources/`.
4. For each significant entity mentioned: create or update its entity page.
5. For each significant concept or theme: create or update its concept page.
6. Update `wiki/index.md` with the new and modified pages.
7. Append an entry to `wiki/log.md`.
8. If this is every 5th+ source, consider updating `wiki/overview.md`.

### Answer a query

1. Read `wiki/index.md` to find relevant pages.
2. Read the relevant pages.
3. Synthesize an answer with `[[wiki-link]]` citations.
4. If the answer is substantial (comparison, analysis, synthesis), offer to file it as an analysis page.
5. If the answer reveals gaps, suggest sources to look for.

### Lint the wiki

Run through this checklist:
- [ ] Orphan pages (no inbound links from other wiki pages)
- [ ] Broken links (links to pages that don't exist)
- [ ] Stale summaries (source count in frontmatter doesn't match actual mentions)
- [ ] Contradictions (pages making conflicting claims — flag, don't resolve)
- [ ] Missing pages (entities/concepts mentioned 3+ times but lacking their own page)
- [ ] Thin pages (pages with less than 2 paragraphs of content)
- [ ] Index sync (all pages listed in index.md, all summaries current)
- [ ] Overview staleness (overview.md doesn't reflect recent ingests)

Report findings and suggest fixes. Ask before making changes.

## Conventions

- Write in clear, direct prose. No filler, no hedging.
- Prefer short paragraphs. The human reads this in Obsidian.
- When sources disagree, present both views and note the tension. Don't pick sides unless the human asks.
- Dates in ISO 8601: YYYY-MM-DD.
- Tags are lowercase, hyphenated: `machine-learning`, `decision-making`.
- Keep the index alphabetized within each section.
- Log entries use the format: `## [YYYY-MM-DD] verb | Title` (verbs: ingest, query, lint, update, create).

## Growing the schema

This document evolves. When you and the human discover a new convention, page type, or workflow that works well, update this file. Note the change in the log.
