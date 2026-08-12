# NEXORA — Agent Context

**Roles (D-088):** Claude (chat) owns `docs/` — design source of truth. Cursor owns all code. Cursor reads docs, never edits them. Claude never edits code.

**Source of truth:** docs/CONCEPT.md (meta-layer), docs/COMBAT-CORE.md (combat spec), docs/ARCHITECTURE.md (systems), docs/DECISIONS.md (append-only journal D-001…D-091), docs/PRODUCTION.md (backlog + prompts).

**Hard rules for code:**
- Engine (`src/engine/*`) is pure TS: no I/O, no randomness (D-064), no LLM. Fully unit-tested.
- LLM never emits numbers — only closed-enum labels (D-051). Numbers live in code/config.
- Secrets only in Vercel env. Never in repo.
- English for code, comments, commits. Conventional commits.
- One prompt = one feature = one commit. Do not refactor beyond the prompt scope.
- Config hypotheses live in `src/engine/config.ts`, tagged `Q-08`.

**Definition of Done:** tests green + Vercel deploy + curl of the published URL confirms the feature.
