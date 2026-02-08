# DASHBOARD.md - Build Cycle Compliance Dashboard
**Version:** 1.0  
**Last Updated:** 2026-02-08  
**Last Audited:** _not yet audited_  
**Current Build Cycle:** _not started_  

---

## How to Use This Dashboard

This is the **living status tracker** for each build cycle. Agents and humans check here first to understand current compliance state.

- Mark `[x]` only with code evidence (files, tests, policies, or logs).
- **New build cycle?** Reset all checkboxes to `[ ]` and update `Last Audited` and `Current Build Cycle` in the header.
- Each row maps to a numbered section in [AGENTS.md](./AGENTS.md). See that file for full checklist details.

---

## Compliance Matrix

| # | Section | Verify | Build | QA | Human |
|---|---------|--------|:-----:|:--:|:-----:|
| 1 | Pre-Build | Domains, folders, tenant model, typed contracts defined | [ ] | [ ] | [ ] |
| 2 | Build Contract | `UnifiedLayout` on all pages, no deprecated patterns | [ ] | [ ] | [ ] |
| 3 | CMS Content | All CMS via `useCmsConfig` + service layer, no direct Supabase | [ ] | [ ] | [ ] |
| 4 | State Mgmt | Auth via `useAuthStore`, tenant context available, no duplicate state | [ ] | [ ] | [ ] |
| 5 | Design Rules | DRY + SOLID + state hygiene applied | [ ] | [ ] | [ ] |
| 6 | FSM | Correct pattern per complexity tier, no impossible states | [ ] | [ ] | [ ] |
| 7 | Observability | Async boundaries emit telemetry, errors normalize to `AppError` | [ ] | [ ] | [ ] |
| 8 | Post Build | `typecheck`/`lint`/`test` green, journeys pass, RLS + tenant isolation verified | [ ] | [ ] | [ ] |
| 9 | Continuous Audit | Runtime signals reviewed, fixes shipped, changelog updated | [ ] | [ ] | [ ] |
| 10 | RLS | `app_config` public read, admin-only write, policies current | [ ] | [ ] | [ ] |
| 10.5 | Multi-Tenant | Tenant isolation enforced, cross-tenant access blocked, 100% RLS coverage | [ ] | [ ] | [ ] |
| 11 | Theme | Extension points untouched, no premature branding | [ ] | [ ] | [ ] |
| 12 | Key Paths | File structure matches contracted paths | [ ] | [ ] | [ ] |

**Column key:** Build = 1st build pass | QA = post-build code review | Human = manual testing

---

## Current Architecture Snapshot

> Update this section after each release. It captures the current state of key architectural decisions.

- **Layout/navigation:** Unified layout system via `src/components/layouts/` (`UnifiedLayout`, `UnifiedSidebar`).
- **CMS:** `app_config` keys `tos_html` and `footer_html`, accessed via `useCmsConfig` + `src/services/cms-config.ts`.
- **State:** Auth in `src/stores/auth-store.ts`, events in `src/stores/events-store.ts`.
- **Data security:** RLS on `app_config` and tenant-scoped tables; tenant isolation enforced by policy.
- **Observability baseline:** Structured async boundary logging + normalized `AppError` contract.
- **Snapshot fields to maintain each release:** Active routes, key tables, edge functions, and open architecture decisions.

---

## Orchestration Notes (For Agents)

### When to Reset the Dashboard
- At the **start of every new build cycle**, reset all checkboxes to `[ ]`.
- Update the `Last Audited` date in the header when an audit is completed.
- Update `Current Build Cycle` with a short label (e.g., "v1.4 feature batch" or "hotfix-auth-rls").

### When to Update the Architecture Snapshot
- After each **release or significant architecture change**.
- When new routes, tables, edge functions, or architectural decisions are added.
- Keep entries concise — this is a quick-reference, not a design doc.

### Verification Workflow
1. Run verification commands: `bun run typecheck`, `bun run lint`, `bun run test`, `supabase db diff`.
2. Walk through each matrix row and mark `Build` column based on code evidence.
3. `QA` column is marked during post-build code review.
4. `Human` column is marked after manual testing of critical journeys.
5. All three columns must be `[x]` before a build cycle is considered complete.

### Metrics to Capture (Post-Build)
- RLS policy coverage: % of tenant-data tables with RLS enabled.
- Average page load time.
- Error rate by category (from `AppError` contract).
- Record in [CHANGELOG.md](./CHANGELOG.md) alongside the version bump.

