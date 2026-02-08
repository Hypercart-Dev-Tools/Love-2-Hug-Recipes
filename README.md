# Lovable to Hugable
Makes your Lovable code into hugable for users.

This doc/system can be used by **non-technical users, rapid prototypers, and senior developers**.

> **Need help?** Start with **[QUICKHELP.md](./QUICKHELP.md)** — it covers common tasks, scenarios, and FAQ with links to detailed rules.

## Who Should Use This System?

This system is for you if you want to build a Lovable app that is:
- ✅ **Easy to maintain** - Clear patterns prevent technical debt
- ✅ **Easy to extend** - New features plug into existing contracts
- ✅ **Easy for future developers to understand** - Self-documenting architecture with compliance checkpoints

**Multi-user (tenant) ready:** This system supports opening your app to friends, family, and other users when you're ready. You don't have to decide now; the multi-user data protection patterns are built in from day one, so you can more easily scale from personal use to multi-user SaaS without a rewrite.

## How to Use This System

### With Lovable AI:

**Step 1: Upload AGENTS.md**   
Upload the AGENTS.md file into Lovable chat before requesting your first draft.

**Step 2: Instruct Lovable to Follow the Guide**   
After uploading, paste this prompt into chat:

```
Please include AGENTS.md in the repo and follow the checklist-driven
architecture in AGENTS.md for all code you generate.
```

**Step 3: Audit First Draft**.
Request a compliance audit against the compliance dashboard:
```
Please audit the code against the Compliance Matrix in DASHBOARD.md
and report any violations.
```
See: [DASHBOARD.md — Compliance Matrix](./DASHBOARD.md#compliance-matrix)

**Step 4: Each Build Cycle**.
Reset checkboxes in [DASHBOARD.md](./DASHBOARD.md) and update the header dates.

**Step 5: When Bugs Pile Up**
If behavior feels inconsistent or bugs keep recurring, re-audit before adding more features:
```
Please audit the current codebase against the Compliance Matrix in
DASHBOARD.md. Flag violations and fix them in priority order
(security > data integrity > UX).
```
This often catches the root cause — a broken contract upstream — faster than chasing individual bugs.

### With Other AI Assistants (Cursor, GitHub Copilot, Augment, etc.):
1. **Setup:** Add AGENTS.md, DASHBOARD.md, and QUICKHELP.md to your project root
2. **During development:** Reference it in prompts: "Follow architecture rules in AGENTS.md"
3. **Before commits:** Verify compliance using [DASHBOARD.md](./DASHBOARD.md) checklist
4. **When breaking rules:** Document in [CHANGELOG.md](./CHANGELOG.md) per [AGENTS.md §13](./AGENTS.md#13-violations---changelogmd-policy)

See [QUICKHELP.md](./QUICKHELP.md) for common tasks or [AGENTS.md §0](./AGENTS.md#0-how-to-use-this-doc) for detailed workflow.

## Documentation

This system is organized into layered documents. Start at the top and go deeper only when needed.

| Document | Purpose |
|----------|---------|
| 📖 **[QUICKHELP.md](./QUICKHELP.md)** | First stop — common tasks, scenarios, and FAQ |
| 📊 **[DASHBOARD.md](./DASHBOARD.md)** | Build cycle compliance status and architecture snapshot |
| 📋 **[AGENTS.md](./AGENTS.md)** | Full architecture rules and checklists |
| 📝 **[CHANGELOG.md](./CHANGELOG.md)** | Version history, violations, and lessons learned |
| 📚 **[REFERENCES.md](./REFERENCES.md)** | Source material for design principles |

## Technical Assumptions

### **Multi-Tenant Architecture**
- **Tenancy Model**: Multi-tenant SaaS with user-level isolation (each user owns their data)
- **Security**: Row-level security (RLS) enforced on all tenant data tables
- **Isolation Strategy**: `auth.uid()` filtering via Supabase RLS policies
- **Shared vs. Tenant Data**:
  - Shared: `app_config` (global CMS content)
  - Tenant-scoped: `monitors`, `events`, `profiles`, user-specific settings
- **Future-Ready**: Architecture supports migration to org-level tenancy if needed

See [AGENTS.md Section 10.5](./AGENTS.md#105-multi-tenant-isolation-checklist) for complete multi-tenant isolation requirements.

### **Tech Stack**
- **Frontend**: React + TypeScript + Vite
- **Backend**: Supabase (database + auth + RLS + edge functions)
- **State Management**: Zustand (with tenant context in `useAuthStore()`)
- **UI Components**: Component-based with unified layout system

## Why This System Works

This architecture guide is built on **proven software engineering principles** that have stood the test of time over decades:

### **SOLID Principles**
- **Single Responsibility**: Each component, service, and store has one clear purpose
- **Open/Closed**: Extend behavior through composition, not modification
- **Liskov Substitution**: Implementations are swappable without breaking contracts
- **Interface Segregation**: Small, focused interfaces prevent unnecessary dependencies
- **Dependency Inversion**: Depend on abstractions, not concrete implementations

### **DRY (Don't Repeat Yourself)**
- One canonical implementation per behavior (one layout system, one auth source, one CMS pattern)
- Service layers eliminate duplicate data access logic
- Typed contracts prevent reimplementation of the same logic

### **State Hygiene**
- Clear separation: local UI state vs. shared state vs. persisted data
- Explicit async state modeling (`idle | loading | success | error`)
- No duplicate writable copies of the same entity

### **Observability & Error Handling**
- Standardized error contracts across all boundaries
- Telemetry at every async operation
- Normalized error categories for consistent UX

### **Continuous Verification**
- [Compliance matrix](./DASHBOARD.md#compliance-matrix) provides measurable checkpoints
- [Violations log](./CHANGELOG.md) creates a learning feedback loop
- Automated + manual checks prevent architectural drift

**Result:** Maintainable, testable, and scalable codebases that AI assistants and human developers can confidently extend without introducing technical debt.

## Getting Started

```bash
# Install dependencies
bun install

# Run development server
bun run dev

# Type checking
bun run typecheck

# Linting
bun run lint

# Tests
bun run test

# Database diff
supabase db diff
```

## License

**Code:** This project's code is available under standard open-source terms.

**Documentation (AGENTS.md, DASHBOARD.md, QUICKHELP.md):**
Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (Creative Commons Attribution 4.0 International)
© 2026 Hypercart DBA Neochrome, Inc.

When sharing or adapting these documents, you must:
- Credit "Hypercart DBA Neochrome, Inc." as the original author
- Provide a link to the CC BY 4.0 license
- Indicate if changes were made
- Not remove attribution notices

---

For help, start with **[QUICKHELP.md](./QUICKHELP.md)**. For full architecture rules, see **[AGENTS.md](./AGENTS.md)**.
