# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Python & Node Conventions

This repo may contain Python, Node, or both. Detect which applies from the manifest files (`pyproject.toml` / `requirements.txt`, `package.json`) before running anything. Don't introduce a new tool if one is already configured.

**Python**
- Use a virtualenv (`.venv`); never install into the global interpreter.
- Prefer `pyproject.toml`; use the project's existing package manager (uv, pip, poetry).
- Format/lint with `ruff` if configured; tests with `pytest`.
- Add type hints to new public functions.

**Node**
- Use the package manager indicated by the lockfile (`pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`). Don't mix them.
- Run scripts via `package.json` (`npm run test`, `npm run lint`) rather than ad-hoc commands.
- Never commit `node_modules/`; commit the lockfile.

**Both**
- Keep secrets in `.env` (gitignored); provide `.env.example`.
- Before claiming done: run the project's tests and linter, and report the actual output.

## 6. LLM Wiki (Karpathy pattern)

Knowledge for this project lives in a persistent markdown wiki that you maintain, instead of re-deriving it from raw documents each time. This file is the **schema** layer.

**Layout**
```
raw/    immutable source documents (articles, papers, notes). Read only; never edit or delete.
wiki/   LLM-owned markdown pages. You write and maintain these.
  index.md   catalog of every page: link + one-line summary, grouped by category
  log.md     append-only chronological record of ingests, queries, lint passes
```

**Page rules**
- One topic/entity/concept per page, `kebab-case.md` filename.
- Start each page with a short summary, then details. Cross-link related pages with `[[page-name]]`.
- Cite sources by their `raw/` path. Mark claims that contradict another page rather than silently overwriting.

**Operations**
- **Ingest**: when a new file appears in `raw/` and I ask you to process it: read it, discuss key takeaways with me, create or update the relevant wiki pages (a single source often touches several), update `index.md`, append a dated entry to `log.md`.
- **Query**: read `index.md` first, then drill into relevant pages and answer with citations to wiki pages. If the answer is a valuable synthesis, offer to save it as a new wiki page.
- **Lint**: on request, check for contradictions between pages, stale claims superseded by newer sources, orphan pages with no inbound links, concepts mentioned but lacking a page, and missing cross-references. Report findings and fix with my approval; log the pass.

**Rules**
- Never modify `raw/`. Never edit wiki pages without updating `index.md` and `log.md`.
- Code and project decisions that matter long-term (architecture choices, conventions, gotchas) belong in the wiki, not only in chat.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.