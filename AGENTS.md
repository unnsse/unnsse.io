# AGENTS.md

Instructions for AI coding agents working in this repository.

# Karpathy behavioral guidelines

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

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, 
and clarifying questions come before implementation rather than after mistakes.

## Setup

## Running the service locally

```bash
hugo server
```

## Code style & quality

## Secrets & configuration

- **Never bake secrets into Docker images or commit them.** Secrets are injected at runtime
  via `.env` files on the EC2 host. Don't add `ENV` lines with credentials/API keys/tokens to
  any `Dockerfile`, and don't create or commit `.env` files.
- Non-secret config (rule tables, label mappings, size filters) belongs in `rules/` as JSON,
  loaded at runtime — not hardcoded as Python literals.

## Documentation & markdown

- **Footer**: every markdown file must end with this exact footer, and nothing may come
  below it:

  ```text
  ---

  > Modern-Day Software Engineering < https://unnsse.io > © 2026
  ```

All new content and subsequent edits go **above** the footer.

## Rule & documentation self-improvement

These rules apply to `.cursor/rules/*.mdc` and project documentation under `docs/`. They are
codified in `.cursor/rules/self-improvement.mdc` (with formatting/structure guidance in
`.cursor/rules/cursor-rules.mdc`) — keep those files and this section in sync.

- **Add a new rule when** a pattern shows up in 3+ files, a recurring bug class could be
  prevented by a rule, code review repeatedly raises the same feedback, or a new
  security/performance pattern emerges in the codebase.
- **Modify an existing rule when** better examples now exist in the codebase, additional
  edge cases are discovered, a related rule has been updated, or implementation details have
  shifted (file moved, function renamed, threshold changed, etc.).
- **Deprecate or remove** rules that no longer apply. Document the migration path for the
  old pattern in the rule before removing it; don't silently delete it.
- Keep rule examples **synchronized with real code** — out-of-date examples are worse than
  no examples. If you change a function/module that a rule cites, update the rule in the
  same change.
- When a code change you are proposing fits one of the triggers above, include the
  corresponding `.cursor/rules/*.mdc` diff in the same change — don't treat it as a
  follow-up. This mirrors the existing rule about updating `run_tests.sh` alongside new
  tests.

## Git workflow

These rules govern *how* a change enters this repository.

Standard branch lifecycle:

```bash
git checkout main
git pull origin main
git checkout -b <new-branch-name>
# ... edit, commit ...
git push -u origin <new-branch-name>
# ... open PR, address review, merge ...
git checkout main && git pull origin main && git branch -d <new-branch-name>
```

Mandatory rules:

- **Never commit directly to `main`.** All changes — code, docs, rules, config — must land
  via a feature branch and a pull request. Direct commits or pushes to `main` are forbidden
  even for one-line fixes.
- **Always sync `main` before branching.** Run `git checkout main && git pull origin main`
  before `git checkout -b <new-branch-name>`. Branching off a stale `main` causes avoidable
  merge conflicts later and produces PRs that test against the wrong base.
- **One logical change per branch / per commit.** Don't mix unrelated work (e.g. rule edits
  alongside a postprocessing refactor and new test fixtures) in a single branch — split
  them so reviews stay focused and reverts stay surgical.
- **Never bypass git hooks** (`--no-verify`, `--no-gpg-sign`, etc.) unless there is a
  specific, documented reason and the team has been notified. Hooks exist to catch the
  things CI would later catch more expensively.
- **Don't amend or rebase commits that have already been pushed** to a branch others may
  have based work on. Amending your own un-pushed (or solo-feature-branch) commits before
  push is fine.
- **Sync `main` again before opening a PR**, and again before merging if `main` has moved
  meaningfully. A green CI on a stale branch is not the same as a green CI on the actual
  merge result.
- **Keep PRs reviewable** — prefer small, focused PRs over large omnibus ones. If a change
  legitimately needs to touch many files (e.g. a rename across the codebase), call that out
  in the PR description so reviewers know what to skim vs. what to read carefully.
- **Delete merged branches** (local and remote) once the PR is in. Stale branches make it
  harder to find the active work.

## Do not memorize secrets or API keys

- Never store or remember secrets, API keys, etc when encountering them
via any type of config files (e.g. .env, .envrc, .properties, xml or yml files, etc).
Never output or write their values, e.g. when referencing them in your output just put the 
key value, for example:

```
You will need to update the $AWS_TOKEN_KEY's variable value located inside config.yml.
```

---

> Modern-Day Software Engineering < https://unnsse.io > © 2026
