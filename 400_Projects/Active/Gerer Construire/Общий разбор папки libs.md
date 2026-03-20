---
type: note
status: active
created: 2026-01-15
updated: 2026-01-15
tags:
  - project
related: []
---

# Общий разбор папки libs

## Context
[Why this note was created / what prompted it]
Для того чтобы вспоминать и перечитывать структуру папки 

## Content
[Main information, thoughts, or description]
### **1. DESIGN-SYSTEM**

Contains reusable UI components and styling following Angular best practices.

### **design-system/components/** (22 UI Components)

Pre-built, publishable Angular components organized by functionality:

- **Form Controls**: `input`, `input-file`, `checkbox`, `select`, `multi-select`, `dropdown`
- **Display**: `accordion`, `avatar`, `box`, `button`, `chip`, `icon`, `tabs`
- **Layout**: `grid`, `header`, `footer`, `control-wrapper`
- **Interactive**: `menu`, `modal`, `side-sheet`, `stepper`, `search-bar`

Each component is structured as an **isolated library** with:

- `ng-package.json` - Publishable Angular package config
- `tsconfig.lib.prod.json` - Production TypeScript config
- project.json - Nx build configuration
- src - Component source code

**Purpose**: These can be published as separate npm packages and reused across projects.

### **design-system/cdk/**

- `forms/` - Custom Form Control Kit: reusable form utilities, validators, and form helpers

### **design-system/global-styles/**

Global SCSS stylesheets and design tokens shared across all components

### **design-system/icons/**

SVG icon library and icon component system

### **design-system/ui-patterns/**

- `search-table-pattern/` - Complete search + table UI pattern combining multiple components

---

### **2. MODULES**

Feature modules organized by domain/business context (feature-driven architecture):

### **modules/core/**

- `auth/` - Authentication logic, login, token management
- `dictionary/` - Shared business dictionaries, lookups, constants

### **modules/shell/**

Application shell/layout - main container, navigation, main routing

### **modules/base/**

Common/shared business logic:

- `application/` - Use cases, app services
- `domain/` - Business entities, interfaces
- `infrastructure/` - API calls, data access
- `presentation/` - Shared presentational logic

### **modules/business/**

Main business domain features (e.g., project management, construction tasks)

### **modules/client/**

Client/customer management features

### **modules/user/**

User profile, preferences, account management

### **modules/location/**

Location/geographic features

---

## **Architecture Pattern**

Your project follows **Nx monorepo + layered architecture**:

DESIGN-SYSTEM (Reusable UI) ↓ MODULES (Feature Modules) ├─ core (shared services) ├─ shell (app layout) └─ business domains (base, business, client, location, user) └─ Each has: application, domain, infrastructure, presentation layers

**Key Benefits**:

- ✅ Isolated, independently testable components
- ✅ Clear separation of concerns (UI vs Logic)
- ✅ Scalable monorepo structure
- ✅ Reusable components publishable as packages

## Key Takeaways
- 
- 
- 

## Links
- [[]] - 
- [[]] - 

## Sources
- 

## Next Steps
[Optional: what to do with this information]
- 
- 

---
**Last Updated**: 2026-01-15