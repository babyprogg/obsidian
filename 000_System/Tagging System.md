# Tagging System

## Tag Structure: category/subcategory

## Categories

### 1. Domain Tags (#domain/)
**Purpose**: What field does this belong to?

- #domain/philosophy
- #domain/code
- #domain/health
- #domain/art
- #domain/learning
- #domain/finance
- #domain/psychology

### 2. Type Tags (#type/)
**Purpose**: What kind of content?

- #type/principle (General truths)
- #type/technique (Specific methods)
- #type/example (Case studies)
- #type/question (Open questions)
- #type/framework (Mental models)

### 3. Status Tags (#status/)
**Purpose**: Note maturity

- #status/seedling
- #status/budding
- #status/evergreen
- #status/archived

### 4. Action Tags (#action/)
**Purpose**: What needs doing?

- #action/develop (Needs more work)
- #action/link (Needs connections)
- #action/apply (Ready to use)
- #action/review (Needs checking)

### 5. Connection Tags (#connect/)
**Purpose**: Cross-domain connections

- #connect/philosophy-code
- #connect/art-engineering
- #connect/health-productivity
- #connect/learning-teaching

### 6. Project Tags (#project/)
**Purpose**: Related to active work

- #project/jarvis
- #project/100soldiers
- #project/learning-app

### 7. Source Tags (#source/)
**Purpose**: Where did this come from?

- #source/book
- #source/article
- #source/video
- #source/experience
- #source/conversation

## Tag Rules

1. **Maximum 5 tags per note**
2. **Use domain + type minimum**
3. **Add connection tags when linking cross-domain**
4. **Review tags monthly**
5. **Merge similar tags quarterly**

## Tag Queries

### All philosophy notes
\`\`\`dataview
LIST
WHERE contains(tags, "#domain/philosophy")
\`\`\`

### Notes needing development
\`\`\`dataview
TABLE status, tags
WHERE contains(tags, "#action/develop")
\`\`\`

### Cross-domain connections
\`\`\`dataview
LIST
WHERE contains(tags, "#connect/")
\`\`\`