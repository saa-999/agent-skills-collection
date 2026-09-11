---
id: github-design
name: github-design
description: Organizes and writes a project's GitHub-facing documentation — README.md, CHANGELOG.md, docs/ explanation files, and .github/ community health files (CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, issue/PR templates). Does not touch source code, does not modify existing files, and never adds or edits in-code comments — it only creates and organizes standalone documentation files. Use when the user asks to write, organize, or clean up a project's README, changelog, docs, or GitHub repo structure.
category: docs
risk: safe
---

## Hard Constraint — Read the Code, Never Write to It

This skill produces documentation only. It never:

* Modifies, refactors, or reformats any source code file.
* Adds, edits, or removes comments inside source code (`//`, `/* */`, docstrings, etc.).
* Changes logic, naming, or structure of anything the user didn't ask to be documented.

It reads the codebase (structure, file names, function signatures, existing comments) purely as **input** to understand what to document — the same way a technical writer reads code without editing it. Every output of this skill is a new or updated file under `README.md`, `CHANGELOG.md`, `docs/`, or `.github/` — nothing else.

If, while reading code to document it, you notice something that looks like an actual bug or an outdated comment describing behavior the code no longer has — do not fix it and do not silently work around it in the documentation by describing the intended behavior instead of the real one. Document what the code actually does, and separately flag the discrepancy to the user under **Out-of-Scope Observations** (see below). Docs must describe reality, not intent.

## When to Use

Triggered by requests like: "write a README for this," "organize the GitHub docs," "explain how this code works in a doc," "set up CONTRIBUTING/issue templates," "update the changelog," "clean up my repo's documentation." Not triggered by requests to fix, refactor, or comment code — that's a different job.

## Scope Control — Build Only What Was Asked

Documentation work has a natural tendency to sprawl: asked for a README, it's tempting to also generate CONTRIBUTING, CODE_OF_CONDUCT, issue templates, and a CHANGELOG because they "belong together." Don't.

* Create only the file(s) the user actually asked for.
* If other related files are missing and would genuinely help (e.g. the user asked for a README but there's no LICENSE and the README references one), mention it as a suggestion in your summary — do not create it without being asked.
* If a task naturally implies more than one file (e.g. "set up the GitHub community health files" implies several), that's fine — the scope was already broad in the request itself. The rule is: match the actual scope of the request, don't silently expand it.

**Order of operations when multiple files are in scope.** Some files reference others — a README's Contributing section links to `CONTRIBUTING.md`, a badge links to `LICENSE`. Write in this order so later files can correctly reference earlier ones instead of linking to something that doesn't exist yet: `LICENSE` → `README.md` → `CONTRIBUTING.md` / `CODE_OF_CONDUCT.md` / `SECURITY.md` → issue/PR templates → `CHANGELOG.md` → `docs/` content.

## Step 1: Project Discovery and Context (do this before writing anything)

Before generating any documentation, understand what you're documenting:

1. **Structure and stack.** List the repository's top-level structure and identify the project type (library, CLI tool, kernel/systems project, web app, etc.) from file extensions, build files (`Makefile`, `CMakeLists.txt`, `package.json`, `Cargo.toml`, etc.), and folder naming.
2. **Existing documentation.** Identify the primary language(s), build/test commands, and any existing partial documentation (an old README, scattered comments, an existing `docs/` folder) so new documentation doesn't contradict or duplicate what's already there.
3. **What already exists in `.github/` and the repo root** (`LICENSE`, `CONTRIBUTING.md`, issue templates) — never silently overwrite an existing file; if one exists, propose an update and show what would change.
4. **Audience and visibility.** Check whether the repository is public or private, and whether it looks like an open-source project (has or expects external contributors) or an internal/private project (a solo or closed-team codebase, like a private kernel project). This changes what's appropriate: a private repo doesn't need a public-facing "why you should use this" pitch or a CODE_OF_CONDUCT aimed at a public community — its README can be more direct and internally-focused. If this isn't obvious from context, ask rather than assume a public open-source tone by default.
5. **Language.** Default to writing documentation in English, matching the convention used by the vast majority of GitHub tooling, badges, and the standard-readme/Diátaxis/Keep-a-Changelog specs this skill follows — unless the user asks for a different language or the existing project documentation is already in one, in which case match it. Once a language is chosen for a repo's docs, keep every generated file in that same language; don't mix.
6. **Terminology baseline.** Note the exact project name, module names, and any established terms as they appear in the actual code and existing docs (capitalization, spelling, abbreviations). Reuse them exactly the same way across every file you generate — a README calling something "AsasNet" and a docs page calling it "Asas Net" is the kind of inconsistency that makes documentation feel unreliable.
7. **Linked skill reports.** Note any linked skills' reports (e.g. a `check-overflow` / `fix-overflow` report) if the documentation task is about explaining a security-relevant module — the explanation should reflect the current, fixed state of the code, not a stale one.
8. **File and folder naming.** Use consistent, lowercase names for generated folders and files (`docs/`, not `Docs/`; `docs/explanation/`, not `docs/Explanation/`). Linux and WSL filesystems are case-sensitive — a folder created as `Docs/` and later referenced as `docs/` in a link or in another tool's config will silently fail to resolve, the same class of mismatch that caused the skill-discovery path issue earlier in this project. Pick one casing convention and apply it everywhere in the same batch of generated files.

## Step 2: README — Structure (based on the standard-readme specification)

Follow this section order (skip sections that don't apply, but keep the ones present in this order):

1. **Title** — the project name, exactly as established in Step 1.
2. **Badges** (optional) — only add a badge for something that actually exists in the repo (a real CI workflow, a real license file, a real published version). Never fabricate a badge pointing at a build/CI system that isn't actually configured — a broken or fake badge is worse than no badge.
3. **Short description** — one or two sentences: what it is and why it exists.
4. **Table of Contents** — for anything long enough to need one.
5. **Background** — the problem this project solves and why it exists, in a few paragraphs. Detailed internals belong in `docs/`, not here.
6. **Install** — exact steps to get it running, in order, using the actual build commands discovered in Step 1.
7. **Usage** — the smallest realistic example that shows the project doing something.
8. **API / Reference** (if applicable) — for libraries; link out to `docs/reference/` for anything detailed rather than inlining everything.
9. **Security Notes** (for a security-relevant project) — a short section, not a full report: what the project's vulnerability-handling process is, link to `SECURITY.md`, and a one-line pointer to any known, currently-accepted limitation (e.g. a finding tracked as **Partially Resolved** in a `fix-overflow` report). Don't bury a known limitation only inside an internal report the README never mentions — a reader deciding whether to rely on this code deserves to know it exists, even briefly.
10. **Contributing** — link to `CONTRIBUTING.md` rather than duplicating its content here. Omit or reword for a private/internal repo where "contributing" means something different than an open-source pull request flow.
11. **License** — name and link to the `LICENSE` file. Omit entirely for a private repo with no license.

Keep the README itself lean — it's an entry point, not the full manual. Anything long or detailed (architecture rationale, an in-depth walkthrough of a subsystem) belongs in `docs/`, linked from the README.

**Multiple languages.** If the project maintains README versions in more than one language, name them per the BCP 47 language tag convention used by the standard-readme spec (`README.md` reserved for English when multiple languages exist, `README.<lang>.md` otherwise, e.g. `README.ar.md`) — don't invent an ad-hoc naming scheme.

## Step 3: CHANGELOG.md (based on the Keep a Changelog specification)

When the user asks for a changelog, or when documenting a release-worthy set of changes:

* Use exactly six standard categories per version: **Added, Changed, Deprecated, Removed, Fixed, Security**. Only include the categories that actually apply to that version — don't pad with empty headings.
* List versions in reverse-chronological order (newest first), each with an ISO 8601 date (`YYYY-MM-DD`) — unambiguous across locales, unlike "March 24th" or "24/03/26".
* **Never paste raw commit messages or `git log` output into the changelog.** A changelog is written for the people using the project, not as a mirror of git history — describe the user-visible effect of a change, not the implementation detail. ("Fixed a crash when the connection pool was empty" — not "fix null check in conn_pool.c:82".)
* Keep an `[Unreleased]` section at the top for changes not yet part of a tagged version.
* If the project uses Semantic Versioning, say so once near the top of the file; if it doesn't, state whatever versioning scheme it does use instead of leaving it ambiguous.
* Security-relevant entries (e.g. a vulnerability closed by the `fix-overflow` skill) belong under the `Security` category — reference the finding at a summary level (what class of issue, not exploit specifics) rather than reproducing the full technical finding; link to the detailed report file instead of inlining it.

## Step 4: Explaining Code Externally — the Diátaxis Framework

When the user wants a doc file that explains how code works, classify what they actually need before writing — each of the four Diátaxis categories is written differently and serves a different reader:

* **Tutorial** — a hands-on lesson for someone learning the codebase for the first time by doing something concrete (e.g. "build and run the pool allocator step by step"). Written as a guided walkthrough, not a reference.
* **How-to guide** — task-focused steps for someone who already knows the codebase and wants to accomplish something specific (e.g. "how to add a new allocator backend"). Assumes competence; skips background explanation.
* **Reference** — dry, exhaustive, factual description of what exists (function signatures, config options, module list) — no narrative, just accurate facts someone can look up.
* **Explanation** — conceptual background: why the code is structured this way, what trade-offs were made, how pieces relate (e.g. "why this project uses a slab allocator instead of malloc directly"). This is where you document *design reasoning*, separate from any tutorial or reference material.

Ask yourself which one the user's request actually is before writing — a request to "explain how X works" is almost always an **Explanation** or **Reference**, not a tutorial. Put each kind in its own file under `docs/` (e.g. `docs/explanation/allocator-design.md`) rather than mixing narrative explanation into a reference table or vice versa — mixing them is the most common way documentation becomes hard to use.

**Diagrams.** For an Explanation doc describing structure, data flow, or state transitions (e.g. how a pool allocator's freelist changes over its lifecycle), a small Mermaid diagram embedded in the markdown file is often clearer than prose alone. Use one when the relationship being described is spatial or sequential — don't add a diagram just for visual decoration where prose already makes the point clearly.

## Step 5: GitHub Community Health Files

When the user wants the repo's GitHub-facing structure set up or cleaned up, these are the standard files (per GitHub's own documentation on community health files) and what each is for:

| File | Purpose |
|---|---|
| `CONTRIBUTING.md` | How to propose changes: branch/PR conventions, coding standards to follow, how to run tests locally. |
| `CODE_OF_CONDUCT.md` | Behavioral expectations for the community — use an established template (e.g. Contributor Covenant) rather than writing one from scratch unless asked. Skip for a private/internal repo unless the user specifically wants one. |
| `SECURITY.md` | How to privately report a vulnerability — critical for a security-relevant project like this one; if the project already has a vulnerability-handling process (e.g. via `check-overflow`/`fix-overflow`), reflect that actual process here rather than a generic template. |
| `.github/ISSUE_TEMPLATE/` | Structured forms for bug reports and feature requests, so issues arrive with the information needed to act on them. |
| `.github/PULL_REQUEST_TEMPLATE.md` | A checklist contributors fill out when opening a PR (what changed, how it was tested, linked issues). |
| `LICENSE` | Only add if the user has told you which license to use. **Never write license text from memory** — legal wording must be exact. Use the official text from a canonical source (e.g. the SPDX license list or choosealicense.com) matching the exact license the user named, verbatim. Never choose a license on the user's behalf. |

These files can live at the repo root or inside `.github/` — GitHub reads either location, but if both exist for the same file, the `.github/` version takes precedence for what's displayed to visitors. Pick one location and be consistent.

## Step 6: Writing Style — Human, Not Templated

Documentation that reads as generated is as unhelpful as documentation that doesn't exist — people skim past it. Apply the same standard used for this repo's commit messages (see the `git-auto` skill):

* Write plain, direct sentences. Avoid reflexive filler words ("leverage," "utilize," "robust," "comprehensive," "seamless") used because they sound impressive rather than because they're the clearest word.
* Don't pad a short answer into a long section just to look thorough. A one-paragraph Background section is correct if that's all there is to say.
* Write from the actual codebase you discovered in Step 1, not from a generic template filled with placeholders — a README that says "This project does X" where X is a real, specific description of what the code does, not a category-level guess.
* Match tone to audience: a `docs/reference/` file is dry and factual; a top-level README's Background section can be a little more narrative, since it's selling the "why" to a new reader — unless Step 1 established this is a private/internal repo, in which case skip the "sales pitch" tone entirely.

## Step 7: Accuracy Check (mandatory before presenting any file)

A documentation file is not done just because it reads well — it must be checked against reality, the same way a code fix must compile before it's considered done.

1. **Verify every command.** Any install/build/run command written in the docs must match what Step 1 actually found in the project's real build files — do not write a command you assumed would work without confirming it against the actual `Makefile`/`package.json`/build script.
2. **Verify every link and path.** Any internal link to another doc file, source file, or section anchor must point to something that actually exists after your edits are applied. A broken internal link in freshly generated documentation is a preventable error.
3. **Code is the source of truth.** If something you're documenting contradicts what the code (or an existing doc) says — the code's actual behavior wins. Do not silently pick one description over the other; tell the user about the discrepancy explicitly so they can decide whether the code or the old documentation was wrong.
4. **No unverified claims.** Don't state a feature, guarantee, or behavior exists unless you actually confirmed it by reading the relevant code or an existing accurate doc. If you can't verify something the user wants documented (e.g. why a design decision was made, and it isn't evident from code, comments, or commit history), ask rather than guess.

## Out-of-Scope Observations

If, while discovering or reading the codebase for documentation purposes, you notice something outside this skill's job — an actual bug, a security concern, an outdated comment describing removed behavior, a missing test — do not fix it and do not silently document around it. List it under "Out-of-scope observations" in your summary so the user can decide whether it needs a `check-overflow`/`fix-overflow` pass or manual attention.

## Handoff to git-auto

Once documentation files are written and accuracy-checked, this skill's job ends — it does not stage or commit anything itself. If the user wants the changes committed, hand off to the `git-auto` skill, which will review the new/changed doc files and write a commit using the `docs:` (or `security:` for a changelog entry tied to a vulnerability fix) type per its own format rules.

## Limitations (be upfront about these)

* Cannot generate accurate screenshots or diagrams of a UI — flag this and let the user add visual assets manually.
* If the codebase is large or unfamiliar, Explanation-type docs may need the user to confirm design rationale that can't be inferred purely from reading code — ask rather than guess at *why* a decision was made if it isn't evident from comments, commit history, or an existing doc.
* If a README, changelog, or doc file already exists, always show what would change before overwriting it — never silently replace existing human-written documentation.