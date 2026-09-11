---
id: git-control
name: git-auto
description: Mastery of Git and professional commit organization — analyzes staged and unstaged changes, writes clear conventional commit messages, and stages files safely. Triggered when the user asks to commit, stage, or describe git changes (e.g. "commit this", "write a commit message", "git add and commit", "stage these files").
risk: safe
category: control
---

## When to Use

Triggered when the user explicitly asks to commit, stage, or describe changes in Git — e.g. "commit this", "write a commit message", "git add and commit", "stage these files". Do not run git commands proactively without such a request.

## Forbidden Actions (never run these without explicit, separate user confirmation in the same message)

These commands rewrite or destroy history, or can silently discard work. Never run them as part of a normal stage-and-commit flow, even if the task seems to call for it:

* `git push --force` / `git push -f` / `git push --force-with-lease`
* `git reset --hard`
* `git rebase` (interactive or not)
* `git commit --amend`
* `git filter-branch` / `git filter-repo`
* `git branch -D` / any forced branch deletion
* `git clean -f` / `git clean -fd`

If a task seems to require one of these, stop and explain why, then ask the user to confirm explicitly before running it. Never infer this kind of confirmation from a general instruction like "clean up the repo" or "fix the history."

## Never Stage or Commit These, Even Under `git add .`

Before staging anything, check for and exclude:

* Secrets and credentials: `.env`, `.env.*`, files containing API keys, tokens, private keys (`*.pem`, `*.key`), passwords, or connection strings.
* Build artifacts and binaries: object files (`*.o`, `*.obj`), compiled binaries, `*.exe`, archives (`*.a`, `*.so`, `*.dll`), unless the user explicitly asks to commit a specific binary.
* Anything already listed in `.gitignore` — respect it, do not override it with `git add -f` unless explicitly asked.

If `git status` shows files matching these patterns as untracked or modified, flag them to the user and exclude them from staging by default — do not silently add them, and do not silently skip mentioning them either.

## Staging Scope

* Default to staging files individually (`git add <file_name>`) after reviewing each one, not `git add .` or `git add -A`.
* Only use `git add .` / `git add -A` if the user explicitly asks to stage everything, and even then, first run `git status` and flag any file matching the **Never Stage** list above before proceeding.

## Git Manager Workflow

Act as an expert Git engineer. Before making any commit, strictly follow this workflow:

1. **Analyze context** — run `git status` to see the current state of the working directory and which files changed.
2. **Review staged changes** — run `git diff --staged` to check what's already prepared for commit.
3. **Review unstaged changes** — for each unstaged file, first read the file itself to understand its context and content, then run `git diff <file_name>` to see the exact modifications. Do not write a commit message from the diff alone without having read the surrounding file.
4. **Check for forbidden content** — apply the **Never Stage** list above to every file before adding it.
5. **Stage** — once the change is understood and cleared, `git add <file_name>` per the Staging Scope rules above.
6. **Commit** — write the message per the format below, then `git commit`.

## Commit Message Format

Use Conventional Commits structure consistently across every commit in this repo:

```
<type>(<scope>): <short summary, imperative mood, under ~72 chars>

<optional body — what changed and why, wrapped at ~72 chars per line>

<optional footer — e.g. references to findings, issues, or breaking changes>
```

Common `type` values: `feat`, `fix`, `security`, `docs`, `refactor`, `test`, `chore`, `perf`.

* Use `security` (not plain `fix`) when the change closes or partially closes a vulnerability finding from `check-overflow` / `fix-overflow` reports. In the body, reference the finding numbers and their resolution status (Resolved / Partially Resolved), matching the status in the report file.
* Keep every commit's format consistent — do not alternate between a detailed style and a one-line style for similar changes. If the user hasn't said which they want, default to the full format above with a short body.
* One logical change per commit. If the staged files represent unrelated changes, say so and suggest splitting into separate commits rather than writing one message that covers both.

## Writing Like a Real Developer, Not an AI

This is the industry-standard reference for commit writing (Chris Beams, "How to Write a Git Commit Message" — the seven rules cited across Git's own documentation, the Linux kernel's contributing guide, and most open-source style guides):

1. Separate the subject from the body with a blank line.
2. Keep the subject line to roughly 50 characters — never let it run on.
3. Capitalize the subject line.
4. Do not end the subject line with a period.
5. Use the imperative mood in the subject ("fix null check", not "fixed null check" or "fixes null check"). Test: the subject should complete the sentence "If applied, this commit will ___".
6. Wrap the body at roughly 72 characters per line.
7. Use the body to explain **what and why**, not **how** — the diff already shows how; the reader needs the reasoning the diff can't show (what problem this solves, why this approach over an alternative, what trade-off was accepted).

### What makes a commit message sound AI-generated (avoid these)

* **Restating the diff instead of explaining it.** "Added a null check to the function" just describes what the diff already shows. A human writes why: "prevent crash when the pool is empty during shutdown."
* **Padding with filler adjectives and hedging.** Words like "leverage," "utilize," "enhance," "robust," "comprehensive," "ensure," used reflexively rather than because they're the most precise word — this is the single most common tell of generated text. Use the plain word: "use" not "utilize," "make sure" not "ensure," if either reads more naturally in context.
* **Listing every changed file exhaustively instead of summarizing intent.** A commit touching five files to fix one bug should describe the fix, not enumerate all five filenames — that's what `git show --stat` is for.
* **Being uniformly verbose regardless of the change's size.** A one-line typo fix needs a one-line message. A subtle concurrency fix earns a real body explaining the race and why this fix closes it. Match the message's length to the change's actual complexity, not to a fixed template.
* **Over-explaining obvious things and under-explaining the actual reasoning.** If the commit reflects a real decision (e.g. choosing a guard-based fix over a struct-layout change because it required zero call-site changes), that reasoning is exactly what belongs in the body — it's the part of your thinking the code itself can't show, and it's what future readers (including future you) actually need.

### Practical rule for this skill

Write the subject line as a plain, direct statement of the change in imperative mood. Only add a body when there's real "why" to explain — a trade-off, a root cause, a limitation, a reference to a finding number. If the change is self-explanatory from the subject alone, don't invent a body just to look thorough; a short, honest commit reads more human than a padded one.