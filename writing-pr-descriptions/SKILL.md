---
name: writing-pr-descriptions
description: Use when writing or rewriting a pull request description — opening a PR with gh pr create, filling in a PR body, editing an existing description, or when a description reads too long. Enforces a short overview-list format with a collapsed AI-decisions block. Keywords: PR description, pull request body, gh pr create, gh pr edit, PR summary, describe this PR, open a PR.
---

# Writing a PR description

**Nobody reads long PR descriptions.** Reviewers read the code. The description exists to point at what changed and to record the few things the code cannot say. Detail beyond that is not thorough — it buries the parts that matter and it is the single most common complaint about AI-written PRs.

Default target: **the visible body fits on one screen.** Roughly 10–15 lines. A large feature PR may reach ~1600 characters; most are far shorter, 150–400 characters.

## Format

```markdown
Closes #119
Iterates #106

Replaces #117, which was cut before the C ABI changed.

# Description

1. Added `pkg/messaging` with an object-oriented `MessagingClient`, mirroring the Nim one.
2. Migrated `internal/ffi` to the new nim-ffi C ABI. `master` of the bindings doesn't compile against logos-delivery `master` anymore, so this is required to unblock CI.
3. Fixed 2 bugs found on the way:
    - the callback's `msg` buffer was read after the callback returned
    - `NIMFFI_RET_STALE_WARN` settled the pending call and fired `wg.Done()` twice
4. Added a compile-only CI step for the tagged integration suite.

<details>
<summary>AI-made decisions</summary>

- Every `Config` field is `omitempty`: the library tells the layered shape from the legacy flat one by looking for bare top-level keys, so emitting zero values would silently fall into the flat parser.
- `Send` races the FFI call against `ctx` cancellation. On cancellation it returns `ctx.Err()`, but the message may still go out.
- README left stale (`libwaku`, `-lwaku`) — belongs with the #106 cleanup.

</details>
```

### Link block

Bare lines at the top, before anything else. `Closes` / `Fixes`, `Iterates`, `Requires`, `Blocked by`, `cherry-pick`.

- **Keyword and reference go on the same line**: `Closes #119`. A `Closes:` heading with the reference on the next line does **not** register as a closing reference — verify with `closingIssuesReferences` if it matters.
- Use `Iterates #N`, never "Part of #N".
- Same-repo issues as `#N`; cross-repo as a full URL.
- Several references under one keyword may be a list, as long as each item carries the keyword.

### `# Description`

A **numbered list, one line per change**, in the order a reader should take them in. Each item says *what was done*, not how it works.

Use a nested bullet list only where several items genuinely belong under one point (e.g. "Fixed 2 bugs found on the way:").

A second sentence on an item is allowed **only** when it explains why this change is in this PR and the code cannot say it — an external constraint, a broken dependency, a compatibility break. Scope it to this repo; downstream impact on other repos is not the reviewer's problem.

### `<details>AI-made decisions</details>`

Optional, collapsed, last. Design choices made without asking — the road not taken, defaults chosen, things deliberately left alone. Not needed to do the review; kept so the reasoning survives in history. One line each.

**Source it from the session, not the diff.** Scroll back through the work that produced the change and pull out the judgment calls: what was tried and abandoned, which default was picked and why, what was knowingly left broken or stale, what an upstream bug or constraint forced. The diff shows the choice that won and never the ones that lost, so this is the only place that reasoning survives.

If nothing non-obvious was decided, omit the block.

### Optional sections

`# Benefits`, `# Usage`, `# References` — only when they carry real content: measured timings, a repro command, a screenshot or video, a list of related links. Never as a place to explain the change again.

## Hard rules

- **No `# Verification` / testing section.** Do not list the commands that passed, the linters run, or the suites that went green. CI reports that.
- **No `# Notes for review` section.** Pre-empting reviewer questions is arguing with them before they speak. If something needs saying, it goes in a code comment or in the `AI-made decisions` block.
- **Do not narrate the code.** No API-surface catalogues, no bolded term/definition bullets, no per-field or per-function explanations.
- **Cut every trailing justification clause.** "so it doesn't rot", "which lets us…", "for better maintainability", and parenthetical inventories like "with unit tests, an integration test and examples".
- **Never hard-wrap lines to a column.** One paragraph is one line; GitHub wraps it.
- **No Claude session links, no "Generated with Claude Code", no `Co-Authored-By: Claude`** — see the global rule in CLAUDE.md. Nowhere public.
- Explanations that belong next to code go in the code, subject to the comment rules in CLAUDE.md — short, one line, only what is genuinely non-obvious.

## Before / after

Same PR (`logos-delivery-go-bindings#120`), 1479 additions across 14 files.

**Before — 4500 chars.** Five `##` sections: a bolded term/definition list of every exported symbol; a section explaining the C ABI change field by field; a `Verification` section listing `go build`, `go vet`, `go mod tidy`, `golangci-lint`, three `go test` invocations with timings, and hand-checked error paths; a `Notes for review` section with four pre-emptive defences.

**After — 1600 chars, 12 visible lines.** The format above. Everything cut was either visible in the diff, reported by CI, or moved into the collapsed block.

## Workflow

1. Read the diff — `gh pr diff <n>` or `git log --oneline <base>..HEAD`. The `# Description` is written from what changed, not from the conversation that produced it.
2. Re-read the session that produced the change, for the `AI-made decisions` block. Decisions exist only in the session: alternatives rejected, defaults chosen, things deliberately left alone, constraints discovered along the way. None of it is recoverable from the diff, and once the session is gone it is gone. If the work spanned a compacted or earlier session, check scratch files and memory too.
3. Check the repo's own conventions: a `.github/pull_request_template.md`, and the title prefix style already in use (`feat:`, `chore_:`, `fix(cmd)_:` — these differ per repo).
4. Draft into a local file, e.g. `~/workspace/pr<n>-body.md`.
5. **Show the draft and get approval before pushing.** Editing a PR description is outward-facing.
6. Push with `gh pr edit <n> --repo <owner/repo> --body-file <file>`, or `gh pr create --body-file <file>`.

## Final check before pushing

Read the draft and delete anything that is:

- visible in the diff,
- reported by CI,
- a justification of why the approach is right,
- a definition of a symbol's API,
- a sentence whose removal loses nothing.

If a reviewer could reply *"this is too long"*, it is. When in doubt, cut.
