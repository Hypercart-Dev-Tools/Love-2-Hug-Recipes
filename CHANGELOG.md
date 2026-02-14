# CHANGELOG

All notable changes and architecture exceptions are documented here.

## Lessons Learned
Use this section for rule violations and follow-ups.

Entry format:
- `YYYY-MM-DD: [Section X] reason -> outcome -> next action #lessonslearned`

Entries:
- 2026-02-14: [v1.6.4] Consolidated OFF-ROAD/STATIC-HOME-PAGE.md into a single self-contained doc with embedded capture scripts, LLM extraction instructions, and Cloudflare Worker setup guide. Documented Lovable hosting constraints (directory indexes don't resolve, SSG not supported) and added TLDR, TOC, and alphabetical appendix sections.
- 2026-02-14: [v1.6.3] Enhanced "Graduation: Spike → Production" section in OFF-ROAD/SUPABASE-TEMP-DB.md with explicit 6-step process. Includes production schema SQL example, TypeScript repository, React Query hook, cleanup commands, and SQL verification queries.
- 2026-02-14: [v1.6.2] Moved SUPABASE-TEMP-DB.md to OFF-ROAD/ folder. Updated references in AGENTS.md, README.md, QUICKHELP.md, and CHANGELOG.md. Added OFF-ROAD/STATIC-HOME-PAGE.md guide for React SPA SEO.
- 2026-02-14: [v1.6.1] Added AI Agent Troubleshooting Protocol to AGENTS.md §12.5. Includes escalation checklist and Minimum Handoff Bundle instructions. Added troubleshooting quick-reference table to README.md.
- 2026-02-14: [v1.6] Added Section 12.5 "Working Outside of Lovable AI Chat" to AGENTS.md. Created comprehensive OFF-ROAD/SUPABASE-TEMP-DB.md guide. Updated README.md with quick-start examples and use cases.
- 2026-02-08: [v1.5] Embedded DASHBOARD.md and QUICKHELP.md templates as appendices inside AGENTS.md with extraction markers and agent instructions (Section 0.0). Single-file delivery: users upload only AGENTS.md; AI agent extracts templates into separate files on first run. Double-trigger pattern: extraction instructions in both AGENTS.md and the user's paste prompt. Updated README.md to reflect single-file upload workflow.
- 2026-02-08: [v1.4] Extracted Compliance Matrix and Architecture Snapshot from AGENTS.md into DASHBOARD.md; created QUICKHELP.md as first-layer help with quick reference and FAQ; updated README.md doc hierarchy to route through QUICKHELP → DASHBOARD → AGENTS; all cross-references use stable intent summaries that won't require edits when target details change.
- 2026-02-08: Initialized violations logging policy and entry format #lessonslearned
