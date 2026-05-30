# AGENTS.md - [Project Name] Architecture Guide (TanStack Start / React / Vite / Lovable Cloud)
**Version:** 1.0
**Last Updated:** [Date]
**Last Audited:** [Not yet audited]
**Purpose:** Canonical architecture rules for **[Project Name]** (TanStack Start + Lovable Cloud). For quick help see [QUICKHELP.md](./QUICKHELP.md). For build cycle status see [DASHBOARD.md](./DASHBOARD.md). For history see [CHANGELOG.md](./CHANGELOG.md).

---

**License:** CC BY 4.0 (Creative Commons Attribution 4.0 International)
**Copyright:** © [Year] [Your Name / Organization]
**Attribution Required:** When sharing or adapting this work, you must:
- Credit "Hypercart DBA Neochrome, Inc." as the original author of the template
- Provide a link to https://creativecommons.org/licenses/by/4.0/
- Indicate if changes were made
- Not remove this attribution notice

**License Terms:** https://creativecommons.org/licenses/by/4.0/legalcode

---

## NON NEGOTIABLE GUIDING PRINCIPLES

Prefer the simplest design that meets the bar — *simple* meaning easy to
understand and change, not least code or fewest files. Security, resilience,
and adequate performance are entry conditions, not things simplicity or
flexibility may override; among designs that clear them, the simplest wins.
When design principles conflict, clarity wins.

Two disciplines keep this balanced — they operate on different axes, not
against each other:
- **YAGNI governs features and speculative abstraction.** Build for today's
  known needs, not hypothetical ones. It does *not* govern decomposition: keep
  units small, well-named, and single-responsibility, and split for clarity as
  soon as it helps understanding — even with one consumer today.
- **Reversibility governs foundations.** Build little — but build it so you can
  change direction. Don't get locked in.

- **SOLID, applied lightly.** Focused responsibilities, intentional
  dependencies, small interfaces. No interface, layer, or abstraction for a
  single *production* caller — unless it is also a necessary test seam, in which
  case the test harness is a real second caller that justifies it.
- **DRY by judgment.** Deduplicate logic that is truly the same and changes
  together. Prefer duplication over a premature or wrong abstraction (rule of
  three before extracting).
- **Stay reversible, don't pre-build.** Spend foresight on decisions costly to
  undo — schema, public contracts, storage/wire formats, framework lock-in —
  and isolate them behind a seam. Distinguish truly one-way doors from merely
  *sticky* ones (hard but not impossible to change); a sticky door rarely earns
  a seam. Everything else is a two-way door: build it simple now, change it
  later. A seam is still attack surface and indirection, so it answers to the
  security and performance floor like any other code.
- **Secure by Design.** Validate at trust boundaries, secure defaults, fail
  closed, defense in depth. Parameterize all queries; never log secrets or PII.
- **Least Privilege.** Every user, service, and component gets the minimum
  access it needs — nothing ambient, nothing "just in case."
- **Prove the floor — don't assume it.**
  - *Tests from day one:* cover behavior with fast, isolated tests; stand up a
    test harness whenever the stack allows.
  - *Observability is part of resilience:* you can't be resilient to failures
    you can't see. Ship meaningful telemetry, structured logging, and health
    checks — enough to detect, diagnose, and recover. Instruments obey the same
    floor: no secrets or PII in logs.
  - *Performance is budgeted:* pin a quantitative budget on any
    performance-sensitive path (p99 latency, throughput, query/row bounds) so
    "adequate" is measured, not assumed. Simplicity operates freely below the bar.

### Applied to this repo ([Project Name])

> **AI Agent Instruction:** Run §0.8 discovery checklist to populate this section before any build work begins.

- **DRY:** [layout primitive and path]; [auth source and path]; [config/CMS pattern]; [one canonical data-access path]. Extract only when the abstraction preserves readability.
- **Single Responsibility:** [what components handle]; [where I/O lives]; [split rule].
- **Open/Closed:** [how to extend without rewriting stable flows — new entries, variants, config keys].
- **Liskov:** [replaceable implementation rule — mocks/adapters must preserve the same contract].
- **Interface Segregation:** [small interface examples — name the hook/service and what it exposes].
- **Dependency Inversion:** [what components depend on instead of raw primitives].
- **State hygiene:** [local vs. shared vs. persisted]; [status union pattern]; [reset rule].

---

## TL;DR) Read This First (Non-Negotiables)
- All pages use `[LayoutComponent]` from `[path]`. Width is set via the `width` prop only — no per-page `max-w-*` overrides.
- Routing is **TanStack Start file-based** under `src/routes/` (NOT `src/pages/`). Never edit `src/routeTree.gen.ts`.
- Auth source of truth is `[authHook]` from `[path]` ([React Context / Zustand / other]). No hardcoded user IDs.
- Server-only **implementation** lives under `src/server/*.server.ts` (never imported from client code). The **RPC wrappers** built with `createServerFn` live under `src/server-fns/*.functions.ts` and are the only server entrypoint that routes/components may import. Never import `client.server.ts` from components.
- All user-data tables have RLS enabled; admin checks use the `has_role()` security-definer function. Never reference `auth.users` via FK — use `profiles`.
- **Dev login** (`src/server-fns/dev-login.functions.ts`) is origin-gated to `*.lovable.app`, `*.lovable.dev`, `*.lovableproject.com`, and `localhost`. Never expand the allow-list.
- Log intentional architectural deviations in `CHANGELOG.md` with `#lessonslearned`.

## 0) How to Use This Doc
- Treat each section as a pass/fail checklist.
- Mark items done only with code evidence (files, tests, policies, or logs).
- Follow the order: pre-build → build contract → post-build → continuous loop.
- **First time in this repo?** Extract the DASHBOARD.md and QUICKHELP.md templates (see Section 0.0 below).
- **Need quick help?** Start with [QUICKHELP.md](./QUICKHELP.md).
- **Checking compliance status?** See [DASHBOARD.md](./DASHBOARD.md).
- **New build cycle?** Reset the dashboard checkboxes and update dates in [DASHBOARD.md](./DASHBOARD.md).

## 0.0) File Extraction (One-Time Setup)

> **AI Agent Instruction:** On first use in a new repo, check whether `DASHBOARD.md` and `QUICKHELP.md` exist in the project root. If they do not exist, extract them from the templates in Appendix A and Appendix B below. After extraction:
> 1. Create `DASHBOARD.md` from the Appendix A template.
> 2. Create `QUICKHELP.md` from the Appendix B template.
> 3. Confirm both files exist before proceeding with any build work.
>
> If both files already exist, skip this step.

## 0.1) Compliance Matrix

The compliance matrix tracks pass/fail status for each architecture section across build, QA, and manual testing phases. It is maintained as a living dashboard in a dedicated file.

→ **[DASHBOARD.md — Compliance Matrix](./DASHBOARD.md#compliance-matrix)**

## 0.2) AI Agent Working Guardrails
- Make one change cluster at a time, verify it, then proceed.
- Edit surgically; do not delete or rewrite files wholesale unless explicitly requested.
- Check existing code before creating new files, hooks, or services.
- If a pattern is unclear, ask: "Which AGENTS.md pattern applies here?"
- Prefer smallest working diff that preserves current contracts.

## 0.3) Dependency and Import Contract
- Before creating a new utility/helper, search existing implementations in `src/services/`, `src/lib/`, and `src/utils/`.
- Keep dependency direction one-way: pages/components -> hooks/services -> integrations.
- Avoid circular imports and cross-feature back-references.
- Reuse public module exports before adding parallel helpers.
- Keep one canonical helper per concern to prevent drift.

## 0.4) Build Break Recovery Protocol
- Capture the failing command and first actionable error.
- Isolate the latest change that introduced failure.
- Fix the smallest root cause first (types/imports/contracts before refactors).
- Re-run the failing command, then rerun the full verification set.
- If unresolved, revert only your latest change chunk and retry with a narrower diff.

## 0.5) Current Architecture Snapshot

The architecture snapshot captures the current state of layout, CMS, state management, security, and observability decisions. It is updated after each release.

→ **[DASHBOARD.md — Architecture Snapshot](./DASHBOARD.md#current-architecture-snapshot)**

## 0.6) Overall Health Grade

The Overall Health Grade is a high-level indicator of the project's architectural health and compliance, derived from the completed compliance matrix in [DASHBOARD.md](./DASHBOARD.md). It provides an at-a-glance summary of the build cycle's quality.

- **Grade A:** Fully compliant; all checklist items passed.
- **Grade B:** Minor, non-critical deviations; requires review.
- **Grade C:** One or more significant deviations; requires immediate fixes.
- **Grade D/F:** Multiple critical violations; build is at risk.

The grade is assessed at the end of each audit cycle and serves as a guide for prioritization. A grade of B- or below signals that technical debt remediation should be prioritized over new features.

## 0.7) Code Intelligence (ask-self RAG)

This repo ships a thin integration layer for **ask-self**, a repository-grounded RAG over this codebase and its docs. The index lives at `temp/rag/[project-name]-self-ask.sqlite` (gitignored). The wrappers live in `scripts/`, the harness lives in `ask_self/`, and the external ask-self install is referenced via `ASK_SELF_PATH` (default `/path/to/ask-self`).

### Who this is for

- **Lovable AI** (the agent that edits this repo from the Lovable Cloud authoring surface) runs inside its own retrieval system and **does not** need ask-self. Lovable should ignore this section. Do not add ask-self steps to Lovable's workflow, do not ask Lovable to run the wrappers, and do not block a Lovable change on a fresh ingest.
- **Local agents** (Claude Code, Cursor, Augment, Copilot Chat, anything driven from a developer's VS Code or terminal) **must** use ask-self. This section is a hard rule for that audience.

### Local-agent rules (non-negotiable)

- **Query first, grep second.** Before grep-spelunking, before walking the file tree, and before asking the user to re-explain anything about this repo, run:
  - `./scripts/ask-self-query.sh "your question here"`
  - Use it for: session-start orientation, unfamiliar subsystems, pronoun-heavy questions ("the auth flow", "that helper", "this dialog"), and any cross-file behavior question.
  - Skip it for: trivial single-file reads, tight edit-test loops you are already inside, and questions about uncommitted state (the index is frozen at the last ingest).
- **Cite from the answer.** ask-self returns `supporting_evidence` with file labels — quote those paths in your reply to the user instead of restating the answer in your own words.
- **Refresh when stale.** The index reflects the last ingest, not the current working tree. Refresh when any of these are true:
  - the timestamp on `temp/rag/[project-name]-self-ask.sqlite` is older than the latest commit on `main`,
  - a structural change has landed since the last ingest (new route, new server function, new Supabase migration, renamed module, deleted file),
  - a query returns "context does not support an answer" for a topic you can see in the working tree,
  - more than ~24 hours of active development have passed without a refresh.
- **Refresh command:**
  - `./scripts/ask-self-ingest.sh` — full refresh, `--mode all`, no PR ingestion.
  - The script auto-prefers `$ASK_SELF_PATH/.venv/bin/python` if present, otherwise falls back to `python3`.
  - `GOOGLE_API_KEY` must be in the environment (Gemini-backed embeddings + synthesis). If a local agent does not have it, ask the operator to export it from their secrets store before retrying — do not silently fall back to grep.
- **Do not commit `ARCHITECTURE.md` blindly.** Ingest regenerates it. If the diff is purely a fresh ask-self regeneration, treat it like a build artifact: commit it deliberately, not as a side-effect of unrelated work.

### Override and escape hatches

- `ASK_SELF_PATH=/some/other/checkout ./scripts/ask-self-query.sh "..."` — point at a different ask-self install.
- `ASK_SELF_PYTHON=/path/to/python ./scripts/ask-self-ingest.sh` — pin a specific Python interpreter.
- If ask-self is unavailable on a given machine, the wrapper fails loudly with a clear path-not-found message — do not paper over the error, surface it to the operator.

### Strong preferred phrasing for local agents

> Before grep-spelunking or asking the user to re-explain repo context, query ask-self first. If the answer cites stale paths, refresh the index, then re-query. Only fall back to manual exploration when ask-self has no relevant context.

## 0.8) Individual Repo Best Practices

> **AI Agent Instruction:** On first use in a new repo, work through the discovery checklist below to derive the project's concrete architectural anchors. Once you have answers, write an "Applied to this repo ([Project Name])" sub-section directly under the NON NEGOTIABLE GUIDING PRINCIPLES section, using the output format below. Do this before any build work begins. If the sub-section already exists, verify it is still accurate against the current codebase and update any stale entries.

The abstract principles in the NON NEGOTIABLE GUIDING PRINCIPLES section need to be grounded in repo-specific, pass/fail rules before they can be applied reliably. This section is a one-time bootstrapping step for each project — run the checklist, write the answers as a named sub-section, then keep it current as the architecture evolves. A stale "Applied to" section is worse than none.

### Discovery Checklist

Read the codebase (or ask the operator) to answer each question. Each answer becomes one bullet in the output sub-section.

| # | Question | Principle it grounds |
|---|----------|----------------------|
| 1 | What is the single layout wrapper component, and where does it live? | DRY (layout) |
| 2 | What is the auth source of truth — hook, store, or context — and where is it defined? | DRY (auth), Dependency Inversion |
| 3 | How do components reach the database or backend? Name the canonical path (server functions, API routes, direct SDK, etc.). | Single Responsibility, Dependency Inversion |
| 4 | What handles cross-route shared state — context, store, or server refetch? | State hygiene |
| 5 | How is dynamic config or CMS content stored and read? | DRY (config) |
| 6 | Where is the server/client boundary drawn, and what enforces it (bundler plugin, linter rule, convention)? | Single Responsibility, Dependency Inversion |
| 7 | What is the one command that must pass before any merge (typecheck + lint + test + build)? | Prove the floor |
| 8 | What is the canonical pattern for async state in components (loading / error / success)? | State hygiene |

### Output Format

Write the sub-section with this shape immediately after the last bullet of the NON NEGOTIABLE GUIDING PRINCIPLES section (before the `---` divider):

```
### Applied to this repo ([Project Name])

These principles translate to concrete, pass/fail rules in this codebase:

- **DRY:** [layout primitive and path]; [auth source and path]; [config/CMS pattern]; [one canonical data-access path].
  Extract only when the abstraction preserves readability.
- **Single Responsibility:** [what components handle]; [where I/O lives]; [split rule].
- **Open/Closed:** [how to extend without rewriting stable flows — new entries, variants, config keys].
- **Liskov:** [replaceable implementation rule — mocks/adapters must preserve the same contract].
- **Interface Segregation:** [small interface examples — name the hook/service and what it exposes].
- **Dependency Inversion:** [what components depend on instead of raw primitives].
- **State hygiene:** [local vs. shared vs. persisted]; [status union pattern]; [reset rule].
```

## 1) Pre-Build Checklist (Before First Feature)
- Define domain boundaries: [list your app's main domains, e.g. auth, data management, admin, shared UI, tenant isolation].
- Define tenant model: single-tenant vs. multi-tenant, user-level vs. org-level isolation, shared vs. tenant-scoped tables.
- Define source of truth: Supabase for persisted data, [state management choice] for shared client state, component state for local ephemeral UI.
- Freeze folder contracts: `src/routes`, `src/components`, `src/server`, `src/server-fns`, `src/integrations/supabase`.
- Design `app_config` keys and RLS before building CMS-driven pages.
- Design RLS policies for tenant data tables before building user-facing features.
- Define typed contracts first (query result types, union states, service interfaces).

## 2) Build Contract Checklist (Current Repo Rules)
- All pages use `[LayoutComponent]` from `[path]` with a `width` variant (`sm | md | lg | xl | full`).
- Page/container `max-width` is defined **once** in `[LayoutComponent]`'s `widthClass` map. To add a width, edit the map — never override at the page level.
- Top navigation lives in `[path/to/Header.tsx]`. [Top-nav app / sidebar app — pick one and do not introduce the other without explicit request.]
- Routes are TanStack Start files under `src/routes/` (flat dot-naming). Never edit `src/routeTree.gen.ts`.
- Server-only logic lives in `src/server/*.server.ts`. RPC wrappers built with `createServerFn` live in `src/server-fns/*.functions.ts` and dynamically import their `.server.ts` sibling **inside `.handler()`**. The auto-generated `src/integrations/supabase/client.server.ts` may only be imported from `src/server/**`.
- [List any existing shared components that must be reused rather than re-implemented — add as the project grows.]

## 2.1) Lovable Dev Login Contract
- RPC wrapper: `src/server-fns/dev-login.functions.ts` (`devLogin`).
- Server-only implementation: `src/server/dev-login.server.ts` (`provisionDevLogin`). Imported dynamically inside the `devLogin` handler — never at module scope.
- Origin allow-list is hard-coded to `*.lovable.app`, `*.lovable.dev`, `*.lovableproject.com`, and `localhost`. Do not expand.
- The dev account email is fixed (`dev@lovable.local`) and its password is rotated on every call.
- The login button on `/login` is rendered only when `window.location.hostname` matches the allow-list.
- The dev login MUST NOT bypass RLS for the resulting session — it signs in as a normal user via `signInWithPassword`.

## 2.2) Server-Function Boundary Contract (CRITICAL)

This is the rule the build will reject you for if violated. The Lovable Vite preset enables TanStack Start's `import-protection` plugin with the pattern `**/server/**` denied in the **client environment**. That means **anything inside `src/server/` is treated as server-only by the bundler**, even a thin `createServerFn` wrapper. RPC wrappers must therefore live OUTSIDE `src/server/`.

### Folder convention

```text
src/server/                 # SERVER-ONLY. Never importable from routes/components.
  *.server.ts               # Implementation (DB, secrets, @tanstack/react-start/server).
  *.types.ts                # Shared types crossing the boundary (no runtime code).

src/server-fns/             # CLIENT-IMPORTABLE RPC wrappers.
  *.functions.ts            # Thin createServerFn() wrappers. NO top-level value
                            # imports from src/server/*.server. Use dynamic
                            # `await import("@/server/x.server")` inside .handler().
                            # Type-only imports from src/server/*.types are fine.
```

### Required wrapper shape

```ts
// src/server-fns/foo.functions.ts
import { createServerFn } from "@tanstack/react-start";
import type { FooResult } from "@/server/foo.types";

export type { FooResult } from "@/server/foo.types";

export const runFoo = createServerFn({ method: "POST" })
  .inputValidator((data: { id: string }) => data)
  .handler(async ({ data }): Promise<FooResult> => {
    const { runFooServer } = await import("@/server/foo.server");
    return runFooServer(data.id);
  });
```

### Do / Don't

- DO put `createServerFn` wrappers in `src/server-fns/`.
- DO dynamically import the `.server.ts` sibling inside `.handler()`.
- DO use `src/server/*.types.ts` for any interface/type that both sides need.
- DO add a status check to `src/server/status.server.ts` for any new server dependency.
- DON'T put a `createServerFn` wrapper inside `src/server/` — import-protection will reject the route file that imports it (build error: `Import denied in client environment ... Denied by file pattern: **/server/**`).
- DON'T top-level-import a `.server.ts` module from a `.functions.ts` wrapper — the static graph analysis still sees the chain even when the runtime is server-side.
- DON'T import `@/integrations/supabase/client.server` outside `src/server/**`. The ESLint rule + Vite plugin both block it.
- DON'T import `@tanstack/react-start/server` outside `src/server/**` or a server-route file under `src/routes/api/`.

### Debugging build failures (for future maintainers)

Symptom: `[tanstack-start-core:import-protection] Import denied in client environment` referencing a file inside `src/server/`.

1. **Read the trace.** It lists every hop from `src/router.tsx` to the offending import. The deepest entry is the file that violated the boundary.
2. **Identify the offender:**
   - If the trace ends at a route file importing `src/server/something`, move that wrapper to `src/server-fns/something.functions.ts` and update the route import.
   - If the trace ends at a `.functions.ts` file importing a `.server.ts` sibling, convert the import to a dynamic `await import(...)` inside `.handler()` and replace any value import with a type-only `import type { ... } from "@/server/x.types"`.
3. **Recheck transitively.** Server-route files under `src/routes/api/*.ts` are allowed to import `@tanstack/react-start/server` because they ship as server handlers. Page route files (`src/routes/foo.tsx`) are not.
4. **Don't disable the plugin.** The Lovable preset is locked; do not add `vite.config.ts` overrides for import-protection. Fix the boundary.
5. **Verify with a real build.** Vite dev mode does not always exercise the rollup `generateBundle` hook that runs import-protection. Always run `bun run build` to confirm the fix, not just `bun run dev`.
6. **If the same `.functions.ts` keeps surfacing different errors,** the file is likely doing too much. Split it: `src/server-fns/x.functions.ts` (RPC), `src/server/x.server.ts` (logic), `src/server/x.types.ts` (shared types).

### Why this convention exists

The Lovable-managed `defineConfig()` in `vite.config.ts` enables import-protection with a folder pattern, not a filename pattern. We can't change the pattern. We can change where the RPC wrappers live. Keeping `src/server/` strictly server-only and `src/server-fns/` as the public RPC surface is the only stable shape.

### Automated enforcement (CI)

The boundary contract is verified on every push and PR by `.github/workflows/ci.yml` → `bun run check:architecture` (script: `scripts/check-architecture.mjs`). The script fails the build if it finds:

- a `*.functions.ts` file inside `src/server/` (must move to `src/server-fns/`),
- a top-level value import of a `*.server` module from a `.functions.ts` wrapper,
- an import of `@/integrations/supabase/client.server` outside `src/server/**`,
- an import of `@tanstack/react-start/server` outside `src/server/**` or `src/routes/api/**`,
- a route/component importing `@/server/*` (must use `@/server-fns/*` or type-only `@/server/*.types`).

Locally, run `bun run verify` to execute the full gate (architecture → typecheck → lint → test → build) before pushing.

## 2.2) UI/UX Conventions

These are app-wide interaction patterns. Use the canonical primitive instead of re-implementing the behavior per page.

> **AI Agent Instruction:** As you build the app, document each reusable UI primitive below. Each entry should name the component, its file path, and its behavior contract. Follow the structure shown. Remove placeholder entries and replace with real ones as the project grows.

### [Canonical primitive — e.g. Click-to-edit text]
- **Canonical primitive:** `[ComponentName]` from `[path]`.
- **When to use:** [describe the use case].
- **Behavior contract:** [key behaviors — save on blur, escape to cancel, etc.].
- **Do not:** [common misuse patterns to avoid].

### Keyboard-first interaction
- Forms submit on Enter; primary input on every screen has `autoFocus`.
- Never trap focus in an element that has no visible exit affordance.
- [Add app-specific keyboard shortcuts here as the project grows.]

### New page QA checklist (MANDATORY before declaring a page done)
Every newly added route under `src/routes/` MUST be QA'd against this list before the agent reports completion. Skipping any item is a §13 violation.

1. **Layout:** wrapped in `[LayoutComponent]` with an explicit `width` variant. No per-page `max-w-*` overrides.
2. **Breadcrumbs:** `breadcrumbs` prop passed if the layout supports it (current page last, no `to`).
3. **Head/SEO:** route exports `head()` with a unique `<title>` (<60 chars) and meta `description` (<160 chars). No reuse of the home page metadata.
4. **Auth gating:** if the route reads user data, it redirects unauthenticated users via the auth hook. Admin routes additionally wait for `roleLoaded` before redirecting.
   - **Async-auth race rule:** Never call `navigate(...)` synchronously after `supabase.auth.signInWithPassword` / `signUp` / `signInWithOtp` resolves. The auth context updates asynchronously via `onAuthStateChange`; a synchronous navigate makes the destination route mount with `user=null` and bounce back to `/login`. Always rely on the destination route's `useEffect([user, loading])` redirect.
5. **Loading / empty / error states:** every async surface renders all three. No bare spinners that never resolve.
6. **Keyboard:** primary input has `autoFocus`; forms submit on Enter; destructive actions confirm.
7. **Server access:** any DB / secret access goes through `src/server-fns/*.functions.ts` (RPC wrapper) backed by `src/server/*.server.ts` (implementation). Never import `client.server.ts` or any `*.server.ts` file from the route file.
8. **Type-check + lint:** `bun run typecheck` and `bun run lint` pass with no new warnings tied to the new file.
9. **Smoke test:** navigate the preview to the new route and confirm first-paint render + one primary interaction.
10. **System status:** new server functions are added as a check in `src/server/status.server.ts` so `/status` reflects the new dependency.

### System status page
- `/status` runs the full self-test suite + Postgres connectivity + RLS leak probes.
- **Cadence:** results are cached for **15 minutes**. Page loads return the cached snapshot — never auto-run on every visit.
- **Admin-only re-run:** the "Re-run now" button is rendered only for admins. The server function enforces admin via bearer-token + `has_role` RPC; non-admin `force=true` is ignored. Do not weaken either side of this check.
- Include outbound links to `https://status.lovable.dev` and `https://status.supabase.com` (open in new tab).

### Changelog viewer
- `/changelog` renders `CHANGELOG.md` with pagination and search/filter. Linked from the footer next to the system-status link.

### Admin gating
- Admin-only pages MUST wait for `roleLoaded` from the auth hook before deciding to redirect. Redirecting on `isAdmin === false` while role is still loading produces a flash-and-bounce.

## 3) CMS Content Checklist (`app_config`)
- [Choose one and delete the other:]
- **Not applicable.** No CMS surface exists. If one is added later, route reads/writes through a `src/services/cms-config.ts` service and a `useCmsConfig` hook, with public read + admin-only write RLS.
- **Applicable.** All CMS content is stored in `public.app_config` as JSON keys. Access via `useCmsConfig` hook and `src/services/cms-config.ts`. Public read, admin-only write enforced by RLS. Keys defined: [list keys here].

## 6) FSM Decision Matrix + Trigger Checklist
| Situation | Recommended Pattern |
|---|---|
| 2 or fewer independent toggles, no invalid combinations | `useState` booleans |
| 3+ mutually exclusive modes, invalid combinations possible | discriminated union + reducer |
| multi-step async flow (retry/cancel/timeout/backoff), guards, role-dependent transitions | explicit FSM (state + event + transition map) |

Switch trigger checklist:
- More than one boolean is needed to represent a single UI mode.
- Impossible states can be represented accidentally.
- Transition rules depend on role, async outcome, or retries/timeouts.

Concrete example (boolean drift -> FSM-style union):
```ts
// Before (easy to create impossible states)
const [isLoading, setIsLoading] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [hasError, setHasError] = useState(false);

// After (single source of truth for mode)
type SaveState =
  | { status: 'idle' }
  | { status: 'saving' }
  | { status: 'success' }
  | { status: 'error'; error: AppError };
```

## 7) Observability + Error Contract Checklist (Tiered)

Baseline tier (required for Lovable-hosted and self-hosted):
- Every async boundary (Supabase call, auth transition, edge function call) emits `start/success/failure` logs.
- Logs include: `feature`, `requestId`, `durationMs`, and `errorCode` on failures.
- A shared app error shape is enforced across UI/store/server boundaries.
- Raw errors are normalized at boundaries before reaching UI components.
- User-facing error handling maps to contract categories (`validation`, `auth`, `permission`, `not_found`, `conflict`, `network`, `db`, `unknown`).

Advanced tier (required when self-hosting or using dedicated telemetry tooling):
- Forward boundary events to a centralized sink (log aggregation/APM).
- Propagate correlation IDs across frontend, edge functions, and Supabase operations.
- Alert on error-rate and latency thresholds by feature/domain.
- Track release-over-release trend lines for error categories and latency.

Recommended contract:
```ts
type AppError = {
  code: string;
  message: string;
  category: 'validation' | 'auth' | 'permission' | 'not_found' | 'conflict' | 'network' | 'db' | 'unknown';
  retryable: boolean;
  httpStatus?: number;
  requestId?: string;
  details?: Record<string, unknown>;
};

type Result<T> = { ok: true; data: T } | { ok: false; error: AppError };
```

## 8) Post First Build Checklist
- Critical journeys pass in local + preview (auth, [list your app's critical journeys], admin CMS edit/publish if applicable).
- Verification commands pass: `bun run typecheck`, `bun run lint`, `bun run test`, `supabase db diff`.
- Security checks pass: RLS on writable tables, admin-only writes verified.
- Multi-tenant isolation verified (cross-ref Section 10.5):
  - User A cannot read/write User B's data
  - Unauthenticated users cannot access any tenant data
  - Cross-tenant queries are blocked by RLS policies
  - Admin access scoped correctly (global config vs. tenant data)
- Resilience checks pass: loading/error/empty states exist for each remote surface.
- `Observability + Error Contract Checklist` (Section 7) baseline tier is passed; advanced tier is passed when self-hosting/telemetry tooling exists.
- `CHANGELOG.md` is updated with version bump and architecture deltas.
- Metrics captured: RLS policy coverage %, avg page load time, error rate by category.

## 9) Continuous Audit -> Fix -> Iterate Checklist
- Audit runtime signals: errors, slow queries, flaky journeys, UX dead-ends, schema/policy drift.
- Prioritize by user impact, security risk, and recurrence.
- Fix in small diffs (one pattern/file cluster at a time).
- Re-verify with typecheck/lint/tests/db diff and re-test touched journeys.
- Record violations/lessons in `CHANGELOG.md` with `#lessonslearned`.
- Repeat at least once per release cycle.

## 4) State Management Defaults Checklist
- Auth source of truth is `[authHook]` from `[path]` ([React Context / Zustand — note if one is intentionally excluded and why]).
- Admin checks use the `isAdmin` boolean from the auth hook, backed server-side by the `has_role()` SQL function.
- Logout uses `signOut()` from the auth hook.
- All Supabase queries are scoped by RLS, not by manual `WHERE user_id = ?` clauses in client code.
- No hardcoded user IDs anywhere.
- Local UI state stays local; cross-route state goes through context or a server-fn refetch.

## 10) Supabase RLS Requirements Checklist
- All tables containing user content ([list your user-data tables, e.g. `items`, `profiles`, `user_roles`]) have RLS enabled.
- Admin gates use the security-definer function `public.has_role(auth.uid(), 'admin')`. Never check roles from a profile/users column.
- Helper functions (`has_role`, and any others you add) are `SECURITY DEFINER` with `search_path = public` and have `EXECUTE` granted to `anon` + `authenticated`.
- [Define whether public read is intentional for your app or whether all reads require auth — document the decision here.]
- Never reference `auth.users` via foreign key — use `public.profiles` instead.

## 10.5) Multi-Tenant Isolation Checklist
- Tenant scope is [**user-level** / **org-level** / **none** — pick one and describe]: [describe how ownership is recorded, e.g. `owner_id` FK to `auth.uid()`].
- Owner-only mutations are enforced by RLS using [describe your helper functions].
- [Describe any collaboration / open-access patterns and how they're RLS-enforced.]
- No org-level tenancy unless explicitly designed. If introduced later, add an `org_id` column + matching RLS before shipping.

## 11) Theme/Branding (Future-Ready) Checklist
- `[LayoutComponent]` is the single source of truth for page/container `max-width` and layout variants.
- `[Header component path]` is the extension point for logo/product/colors.
- [This is a top-nav / sidebar app — do not introduce the other pattern without explicit request.]
- **Site name / brand label must never be hardcoded in UI strings, `<title>` tags, meta descriptions, JSON-LD, or fallback constants.** Always source it from a programmatic token (e.g., a build-time `VITE_SITE_NAME` env var or `app_config` key). If the token is unavailable, use a generic fallback (e.g., "Site") — never a specific product name. This prevents stale-brand flashes during hydration and allows re-branding without code changes.

## 12) Key Paths
- Routes (TanStack Start, file-based): `src/routes/`
- Auto-generated route tree (DO NOT EDIT): `src/routeTree.gen.ts`
- Layouts: `src/components/layouts/`
- Shared UI primitives (shadcn): `src/components/ui/`
- Domain components: `src/components/`
- Auth context: `[path to auth hook/context]`
- App-level helpers: `src/lib/`
- Server-function RPC wrappers (client-importable): `src/server-fns/*.functions.ts`
- Server-only implementation (NEVER client-importable): `src/server/*.server.ts`
- Shared types crossing the boundary: `src/server/*.types.ts`
- Supabase clients (auto-generated, DO NOT EDIT): `src/integrations/supabase/{client,client.server,types,auth-middleware}.ts`
- Migrations: `supabase/migrations/`
- Edge functions (if any): `supabase/functions/`

## 12.5) Working Outside of Lovable AI Chat (Dev Testing Workflow)

> **AI Agent Instruction:** When a user mentions testing Supabase features, prototyping data structures, or validating RLS policies outside of Lovable AI chat, suggest setting up the reusable `dev_sandbox` table pattern. Offer to run the one-time setup wizard (see below) if the table doesn't exist yet.

When developing and testing features outside the Lovable AI chat environment, you need a safe, repeatable way to test Supabase interactions without deploying edge functions or polluting production tables.

### One-Time Setup Wizard

> **AI Agent Instruction:** If `temp_dev_records` table does not exist and user is working on Supabase testing/prototyping, offer to run this setup:
> 1. Ask: "Would you like me to set up the reusable Temp Dev Sandbox for testing? This will create two tables (`temp_dev_records`, `temp_dev_reset_log`) with RLS policies, an edge function for external access, and optional React components. See [OFF-ROAD/SUPABASE-TEMP-DB.md](./OFF-ROAD/SUPABASE-TEMP-DB.md) for details."
> 2. If yes, execute the SQL migration from OFF-ROAD/SUPABASE-TEMP-DB.md (table schema + RLS policies + triggers).
> 3. Optionally deploy the edge function at `supabase/functions/temp-dev-sandbox/index.ts`.
> 4. Optionally copy app-side files: `TempDevRepository.ts`, `useTempDevSandbox.ts`, `TempDevSandbox.tsx`.
> 5. Verify table creation with `supabase db diff` or equivalent.
> 6. Remind user: "Sandbox is ready! Use `feature_key` to scope each spike. Never use this for production data."

### Reusable Sandbox Table Pattern:
- Create two tables: `temp_dev_records` (flexible multi-type fields) and `temp_dev_reset_log` (audit trail).
- Use `feature_key` to scope each spike/experiment (e.g., `kanban_insert_order`, `md_editor_preview_experiment`).
- Support multiple data types: `text_value`, `number_value`, `bool_value`, `json_value`, `tags[]`.
- Provide two access paths: **Direct SDK** (fastest for React) and **Edge Function** (for curl, external LLMs, CI scripts).
- Implement reset utilities: reset by feature key, reset all, purge expired records.
- Keep sandbox separate from production schema; never reference it in production code.

### Checklist:
- `temp_dev_records` table exists with flexible schema (id, user_id, feature_key, text/number/bool/json values, tags, status, sort_index, expires_at).
- `temp_dev_reset_log` table exists for audit trail (user_id, feature_key, action, reason, deleted_count).
- RLS policies enforce user isolation (`auth.uid() = user_id`).
- Edge function deployed at `/functions/v1/temp-dev-sandbox` for external access (optional).
- App-side repository/hook/UI components available for in-app spikes (optional).
- All sandbox usage is documented in `OFF-ROAD/SUPABASE-TEMP-DB.md`.
- Developers know when to use sandbox vs. creating proper test tables.

### When to Use Sandbox vs. Proper Tables:
- **Use sandbox for:** Quick prototyping, testing RLS patterns, experimenting with JSONB queries, validating auth flows, feature spikes, drag-drop ordering tests, cache tuning experiments.
- **Use proper tables for:** Feature development, integration tests, data that needs migrations, production-like testing, user-facing features.

### Graduation Path (Spike → Production):
When a spike proves out and is ready for production:
1. Create a dedicated table with proper migration (typed columns, constraints, indexes).
2. Create a dedicated repository in `src/repositories/`.
3. Create a dedicated hook in `src/hooks/`.
4. Clean up: run `DELETE ?action=reset_feature&feature_key={key}` to remove spike data.
5. Document: log the graduation in `CHANGELOG.md`.

### AI Agent Context Awareness:
> **AI Agent Instruction:** If you detect the user is working on Supabase testing tasks and `OFF-ROAD/SUPABASE-TEMP-DB.md` exists in your context window, proactively suggest using the sandbox table pattern instead of creating temporary tables or deploying edge functions.

### AI Agent Troubleshooting Protocol:
> **AI Agent Instruction:** When sandbox setup or testing fails, use the escalation checklist from OFF-ROAD/SUPABASE-TEMP-DB.md "Lovable Back-and-Forth Scenarios" section:
> 1. **Identify the symptom** (404, 401, RLS errors, schema cache miss, etc.)
> 2. **Diagnose likely cause** from the troubleshooting table
> 3. **Provide specific fix** (not generic "check your config")
> 4. **If escalating to Lovable**, prepare the Minimum Handoff Bundle:
>    - Exact failing curl command + response JSON
>    - Project ref and function URL
>    - Expected migration filename
>    - Whether failure is edge-only or also direct SDK
>    - Timestamp of latest deploy/migration attempt
> 5. **Avoid slow back-and-forth** by including all diagnostic info in one message

→ **Full guide:** [OFF-ROAD/SUPABASE-TEMP-DB.md](./OFF-ROAD/SUPABASE-TEMP-DB.md)

## 13) Violations -> `CHANGELOG.md` Policy
Do not log rule violations in this file. Log each violation and lesson in `CHANGELOG.md`.

**Instructions:**
- Add a changelog entry whenever a rule from Sections 0-12 is intentionally broken.
- Include `#lessonslearned` in each entry.
- Include: date, violated section, business reason, technical outcome, and fix/next action.
- Review `#lessonslearned` entries quarterly and convert repeated patterns into checklist updates.

**Entry template (in `CHANGELOG.md`):**
`- YYYY-MM-DD: [Section X] reason -> outcome -> next action #lessonslearned`

## 14) Revision History

**All version history and change details are maintained in [CHANGELOG.md](./CHANGELOG.md).**

This keeps AGENTS.md focused on current architecture rules and reduces context overhead. For historical context, see CHANGELOG.md.

---

## 15) Breaking Change Management Checklist

**Definition:** A breaking change is any modification that causes existing consumers (UI components, services, hooks, RLS policies, edge functions, or external integrations) to fail without code changes on their side.

### Severity Tiers

| Tier | Scope | Examples | Handling |
|------|-------|----------|----------|
| **Critical** | Schema-level | Dropped/renamed columns, changed column types, removed tables | Requires migration script, rollback plan, and team sign-off before merge |
| **High** | API/contract-level | Changed service interface signatures, modified hook return shapes, altered edge function request/response contracts | Requires deprecation period or versioned endpoint |
| **Moderate** | UI contract-level | Renamed `app_config` keys, changed component prop contracts, altered state hook or store action signatures | Requires backward-compatible shim for one release cycle |

### Identification Checklist
- Evaluate all changes for breaking impacts before opening a PR, especially when:
  - Modifying database tables, columns, types, or constraints.
  - Changing service interfaces, hook return types, or state hook/store action signatures.
  - Updating shared libraries, API contracts, or edge function request/response shapes.
  - Fixing bugs in ways that alter established behavior consumers may depend on.
  - Touching RLS policies that other queries or services rely on (cross-ref Sections 10/10.5).

### Pre-Merge Gate
- Run `bun run typecheck && bun run lint && bun run test` against the existing test suite before merging any schema/API change.
- Verify RLS policies still function correctly after any schema change (cross-ref Sections 10/10.5).
- Confirm no existing consumers break by searching for all call sites of the changed contract.

### Migration & Rollback
- Document a rollback plan: how to revert the change if it fails in production.
- For schema changes: provide a forward migration and a reverse migration script.
- For renamed keys (e.g., `app_config`): keep the old key readable for one release cycle and add a deprecation log entry before removing it.
- For changed service interfaces: support both old and new signatures for one release cycle when feasible.

### Communication & Logging
- Add a `## Breaking Changes` subsection to the CHANGELOG.md release entry.
- Include: affected tier, what changed, migration steps for consumers, and rollback instructions.
- Communicate the impact and migration plan to the team before merging.
- Avoid breaking changes unless there is a clear, documented business need.

---

## 16) Technical Spike & Proof of Concept (POC) Checklist
- **Define Purpose:** Before starting a spike, define a clear, timeboxed objective. Spikes should be used to:
  - Validate the feasibility of a new feature or architectural choice.
  - Identify risks and unknowns early in the planning phase.
  - Minimize surprises during full implementation.
- **Follow Guidelines:**
  - Define clear, measurable objectives for the spike.
  - Timebox the effort to avoid scope creep and analysis paralysis.
  - Use the `temp_dev_records` sandbox for safe, isolated testing (see Section 12.5).
  - Document all findings and decisions in the `OFF-ROAD/` directory or relevant project documentation.

---

## Appendix A: DASHBOARD.md Template

> **AI Agent Instruction:** If `DASHBOARD.md` does not exist in the project root, create it with the content below (everything between the `<!-- BEGIN DASHBOARD TEMPLATE -->` and `<!-- END DASHBOARD TEMPLATE -->` markers). Do not include the marker comments in the extracted file.

<!-- BEGIN DASHBOARD TEMPLATE -->

# DASHBOARD.md - Build Cycle Compliance Dashboard
**Version:** 1.0
**Last Updated:** [Date]
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
| 0.8 | Repo Anchors | "Applied to this repo" sub-section exists, is accurate, and matches current codebase | [ ] | [ ] | [ ] |
| 1 | Pre-Build | Domains, folders, tenant model, typed contracts defined | [ ] | [ ] | [ ] |
| 2 | Build Contract | Layout component on all pages, single `max-width` definition, server boundary clean, no deprecated patterns | [ ] | [ ] | [ ] |
| 3 | CMS Content | All CMS via service layer + canonical hook, no direct Supabase client in components | [ ] | [ ] | [ ] |
| 4 | State Mgmt | Auth via canonical hook, no hardcoded user IDs, no duplicate state | [ ] | [ ] | [ ] |
| 6 | FSM | Correct pattern per complexity tier, no impossible states | [ ] | [ ] | [ ] |
| 7 | Observability | Async boundaries emit telemetry, errors normalize to `AppError` | [ ] | [ ] | [ ] |
| 8 | Post Build | `typecheck`/`lint`/`test` green, critical journeys pass, RLS + tenant isolation verified | [ ] | [ ] | [ ] |
| 9 | Continuous Audit | Runtime signals reviewed, fixes shipped, changelog updated | [ ] | [ ] | [ ] |
| 10 | RLS | User-data tables have RLS, admin gates use `has_role()`, no `auth.users` FK, policies current | [ ] | [ ] | [ ] |
| 10.5 | Multi-Tenant | Tenant isolation enforced, cross-tenant access blocked, 100% RLS coverage | [ ] | [ ] | [ ] |
| 11 | Theme | Extension points untouched, site name sourced from token, no hardcoded brand | [ ] | [ ] | [ ] |
| 12 | Key Paths | File structure matches contracted paths | [ ] | [ ] | [ ] |

**Column key:** Build = 1st build pass | QA = post-build code review | Human = manual testing

---

## Current Architecture Snapshot

> Update this section after each release. It captures the current state of key architectural decisions.

- **Layout/navigation:** [Layout component and nav style — e.g. top-nav, sidebar.]
- **CMS:** [CMS approach and config keys in use, or "not applicable".]
- **State:** [Auth hook/context path and any shared state stores.]
- **Data security:** RLS on [list tables]; tenant isolation enforced by policy.
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

<!-- END DASHBOARD TEMPLATE -->

## Appendix B: QUICKHELP.md Template

> **AI Agent Instruction:** If `QUICKHELP.md` does not exist in the project root, create it with the content below (everything between the `<!-- BEGIN QUICKHELP TEMPLATE -->` and `<!-- END QUICKHELP TEMPLATE -->` markers). Do not include the marker comments in the extracted file.

<!-- BEGIN QUICKHELP TEMPLATE -->

# QUICKHELP.md - First-Layer Help & FAQ
**Version:** 1.0
**Last Updated:** [Date]

Start here before diving into the full architecture guide. Most common tasks and questions are answered below with links to detailed rules when needed.

---

## Quick Reference — "I need to..."

### Add a new page
All pages use a single unified layout system. Follow the build contract for page structure.
→ [AGENTS.md §2 — Build Contract](./AGENTS.md#2-build-contract-checklist-current-repo-rules)

### Set or update page/screen width
Page `max-width` is controlled from a single place. Never add per-page width overrides.
→ [AGENTS.md §2 — Build Contract](./AGENTS.md#2-build-contract-checklist-current-repo-rules)
→ [AGENTS.md §11 — Theme/Branding](./AGENTS.md#11-themebranding-future-ready-checklist)

### Add or edit CMS content
CMS content is managed through a dedicated service layer and hook pattern, not direct database calls.
→ [AGENTS.md §3 — CMS Content](./AGENTS.md#3-cms-content-checklist-app_config)

### Check build cycle compliance
The compliance dashboard tracks pass/fail status for each architecture section per build cycle.
→ [DASHBOARD.md — Compliance Matrix](./DASHBOARD.md#compliance-matrix)

### Fix a broken build
Follow the recovery protocol: isolate the change, fix the smallest root cause, re-verify.
→ [AGENTS.md §0.4 — Build Break Recovery](./AGENTS.md#04-build-break-recovery-protocol)

### Handle authentication or user state
Auth has a single source of truth via the canonical auth hook. All tenant context flows from there.
→ [AGENTS.md §4 — State Management](./AGENTS.md#4-state-management-defaults-checklist)

### Add security policies for user data
All user-facing data requires row-level security policies before shipping. Tenant isolation is enforced at the database level.
→ [AGENTS.md §10 — RLS](./AGENTS.md#10-supabase-rls-requirements-checklist)
→ [AGENTS.md §10.5 — Multi-Tenant Isolation](./AGENTS.md#105-multi-tenant-isolation-checklist)

### Manage complex UI state
Choose the right state pattern based on complexity — simple booleans, discriminated unions, or full state machines.
→ [AGENTS.md §6 — FSM Decision Matrix](./AGENTS.md#6-fsm-decision-matrix--trigger-checklist)

### Add error handling or observability
Errors follow a standardized contract across all boundaries. Async operations emit structured telemetry.
→ [AGENTS.md §7 — Observability](./AGENTS.md#7-observability--error-contract-checklist-tiered)

### Log a rule violation
Intentional rule breaks are logged in the changelog with context and follow-up actions.
→ [AGENTS.md §13 — Violations Policy](./AGENTS.md#13-violations---changelogmd-policy)

### Understand the current architecture
The architecture snapshot captures the current state of key decisions and is updated each release.
→ [DASHBOARD.md — Architecture Snapshot](./DASHBOARD.md#current-architecture-snapshot)

---

## Document Map

| Document | Purpose | When to check |
|----------|---------|---------------|
| **[QUICKHELP.md](./QUICKHELP.md)** (this file) | First-layer help, common tasks, FAQ | First stop for any question |
| **[DASHBOARD.md](./DASHBOARD.md)** | Build cycle status and architecture snapshot | During and after each build cycle |
| **[AGENTS.md](./AGENTS.md)** | Full architecture rules and checklists | When you need detailed implementation rules |
| **[CHANGELOG.md](./CHANGELOG.md)** | Version history, violations, lessons learned | After changes or when reviewing history |
| **[REFERENCES.md](./REFERENCES.md)** | Source material for design principles | When you want to understand *why* a rule exists |

---

## Scenarios & FAQ

### "The AI agent keeps going in circles"
This usually means the agent can't find the right pattern to apply. Point it to a specific AGENTS.md section:
`"Follow AGENTS.md §[number] for this task."`
If the issue persists, check whether the task conflicts with an existing contract.

### "I'm not sure which state pattern to use"
Start with the simplest option that avoids impossible states. The FSM decision matrix provides clear escalation triggers.
→ [AGENTS.md §6](./AGENTS.md#6-fsm-decision-matrix--trigger-checklist)

### "My RLS policy isn't working"
Verify the policy exists and uses the correct auth context. The multi-tenant checklist includes specific test scenarios for cross-tenant access.
→ [AGENTS.md §10.5 — Testing & Verification](./AGENTS.md#105-multi-tenant-isolation-checklist)

### "Should I create a new file or edit an existing one?"
Check existing implementations first. The dependency contract requires reusing existing services and helpers before creating new ones.
→ [AGENTS.md §0.3 — Dependency Contract](./AGENTS.md#03-dependency-and-import-contract)

### "How do I extend the layout or add branding?"
The layout system has designated extension points. Don't add new layout patterns without explicit need.
→ [AGENTS.md §11 — Theme/Branding](./AGENTS.md#11-themebranding-future-ready-checklist)

### "When do I reset the compliance dashboard?"
At the start of every new build cycle. Reset all checkboxes and update the header dates.
→ [DASHBOARD.md — Orchestration Notes](./DASHBOARD.md#orchestration-notes-for-agents)

### "Where do I log bugs or architecture exceptions?"
In the changelog, using the standard entry format with `#lessonslearned` tag.
→ [AGENTS.md §13](./AGENTS.md#13-violations---changelogmd-policy) | [CHANGELOG.md](./CHANGELOG.md)

<!-- END QUICKHELP TEMPLATE -->
