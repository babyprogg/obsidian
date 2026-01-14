# Properties Guide

## Core Properties (All Notes)

### type
**Values**: atomic | structure | literature | project | daily | question | fleeting | area
**Purpose**: Note classification
**Required**: Yes

### status
**Values**: seedling | budding | evergreen | archived
**Purpose**: Note maturity
**Meaning**:
- seedling: New, undeveloped (<3 links)
- budding: Developing (3-10 links, being refined)
- evergreen: Mature, reliable (10+ links, well-developed)
- archived: No longer active

### created
**Format**: YYYY-MM-DD
**Purpose**: Creation date
**Required**: Yes

### updated
**Format**: YYYY-MM-DD
**Purpose**: Last significant update
**Required**: For evergreen notes

## Type-Specific Properties

### For Atomic Notes
- **domain**: philosophy | code | health | art | learning | finance
- **tags**: Relevant tags
- **related**: List of related note links

### For Literature Notes
- **author**: Author name
- **finished**: YYYY-MM-DD
- **rating**: ⭐⭐⭐⭐⭐
- **source**: Book | Article | Video | Podcast

### For Project Notes
- **deadline**: YYYY-MM-DD
- **next-action**: What to do next
- **status**: planning | active | paused | completed | archived

### For Question Notes
- **status**: open | researching | answered | archived
- **priority**: high | medium | low

### For Daily Notes
- **mood**: Great | Good | Okay | Rough | Bad
- **energy**: High | Medium | Low
- **focus-area**: What you're focusing on

## Example Note with Properties

\```markdown
---
type: atomic
status: evergreen
domain: learning
created: 2026-01-11
updated: 2026-01-11
tags:
  - deliberate-practice
  - skill-acquisition
  - mastery
related:
  - "[[Flow State]]"
  - "[[10000 Hour Rule]]"
  - "[[Feedback Loops]]"
---

# Deliberate Practice Requires Specific Goals

[Note content here]
\```