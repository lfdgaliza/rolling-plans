---
name: rp
description: Work with Rolling Plans, one markdown file per feature with requirements R1…Rn. Use when the user runs /rp, or asks to mark a requirement completed, find the next requirement, add or drop a requirement, or check a plan against the code.
argument-hint: "[help | mark Rn completed [PR #n] | next | add <requirement> | drop Rn | check] [plan name or path]"
---

# Rolling Plans

The user follows the Rolling Plans methodology. The rules below are what you need for each action. If the methodology document is available (`README.md` at the root of https://github.com/lfdgaliza/rolling-plans, or a local clone of it), it is the authoritative version: read it, and follow it where it differs from this file.

Request: `$ARGUMENTS`

If the request is `help` (or `?`, or asks how to use `/rp`), skip everything below and reply with exactly this text. Don't read or look for a plan:

```
/rp [action] [plan name or path]

  /rp                         Status: completed, next, remaining
  /rp help                    This help
  /rp mark R1 completed PR #n Rewrite R1 as-built from the PR diff, add ✅ and the PR
  /rp next                    The next requirement, checked against the code
  /rp add <requirement>       Add a planned requirement at the end
  /rp drop R3                 Remove a planned requirement and record a decision
  /rp check                   Report plan/code mismatches and plan problems (no edits)

Plan: name it (/rp next externalCvrValidation), or Claude uses the one in the
conversation, or finds it under plans/ and asks if several fit.
Free text works: "/rp mark r1 as completed".
Edits are shown first and applied only after you confirm.
```

## 1. Find the plan

In this order:

1. A plan name or path in the request (e.g. `/rp next externalCvrValidation`).
2. A plan already read or mentioned in this conversation.
3. A `plans/` folder in the current directory or its parents. If exactly one plan matches the request, use it.
4. Otherwise, list the candidates and ask. Never guess.

Read the whole plan before doing anything else.

## 2. Plan conventions

- Sections, in order: Title, Context, Goal, Decisions, Acceptance, optional sections (Scope, Security considerations, Key files, custom), then Requirements last.
- Each requirement is a `### Rn.` heading, either:
  - **Completed:** `### ✅ Rn. <title> (COMPLETED, PR #1234)`. The body is written as-built, in past tense, and describes what exists in the code now.
  - **Planned:** `### Rn. <title>`. The body is in future tense: context, proposed implementation, acceptance. It says what to do and what not to do (`Do:` / `Don't:` lines), for each part when it has several (endpoints, components, sections of a doc).
- The order of requirements is a timeline: completed ones first, then planned ones. The next requirement is the first one without ✅. There is no "in progress" marker.
- Completed requirements are history and don't move. Planned ones can be reordered, rewritten or removed.
- Decisions are a numbered list stated as they stand now, with a reason when it isn't obvious. When a decision changes, rewrite it instead of striking it through.
- Anything that must be done is a requirement or a checkbox inside one, never only prose in another section.
- Notes (`> Note:`, `> To verify:`) are temporary. Once confirmed, fold them into the text and delete them.
- Key files live inside the requirement they belong to. Once that requirement has a PR link, its key files list can go.

## 3. Actions

Work out the action from the request. Free text is fine (`mark r1 as completed` = `mark R1 completed`).

### mark Rn completed [PR #n]

1. **Find what was built.** Use the PR number from the request; if there isn't one, ask for it. Read the PR diff (`gh pr diff`, or `git log`/`git diff` in the code repo). If you can't reach the code, ask the user where it is. Don't write the as-built version from the plan text.
2. **Rewrite the body as-built**: past tense and factual ("`X` was added to `Y`"). Only describe what you saw in the code. Leave out anything you couldn't verify.
3. **Update the heading:** `### ✅ Rn. <title> (COMPLETED, PR #n)`.
   - Several PRs or services: add a header block under the heading (`> **Service:**`, `> **Branch:**`, `> **PRs:** #1 (backend), #2 (frontend)`).
   - No code needed (e.g. manual configuration): say so in place of the PR.
4. **Divergence → decision.** If what was built differs from the plan in a way that matters, propose a decision for it. If the reason isn't clear from the code or the conversation, ask for it; don't make one up.
5. **Clean up:** resolve that requirement's notes, and propose removing its key files list.
6. **Check the order.** If Rn isn't the first planned requirement, it must move up to sit right after the last completed one. Say so; don't renumber without asking.
7. Mention it if Context or Acceptance now reads stale.

### next

Name the first requirement without ✅ and summarize what it asks for and what it rules out. Check it against the code: if the code contradicts the plan, or part of it already exists, stop and say so instead of quietly picking one. Don't start implementing unless asked.

### add <requirement>

Add a planned requirement at the end of Requirements (or where the user says), numbered one higher than the current highest. Future tense: context, proposed implementation with `Do:` and `Don't:` lines (for each part, if it has several), acceptance. If the request is vague, or you can't tell what it should rule out, ask before drafting the details.

### drop Rn

Only planned requirements can be dropped. Delete the requirement and add a decision saying why and where the work lives now, if anywhere (e.g. "Exports are handled in `other-plan.md` R2"). Ask for the reason if it wasn't given.

### check

Report problems; don't fix them unless asked:

- Completed requirements that don't match the code, or have no PR (and no "no code needed").
- Planned requirements the code already contradicts or already implements.
- Planned requirements with no `Don't:` lines.
- Order violations (a planned requirement above a completed one).
- Work mentioned only in prose sections, with no requirement.
- Unresolved notes, stale or undated facts in Context, a plan-wide key files list.

### No action given

Show the status: completed requirements (one line each), the next one, and the rest.

## 4. Before saving

Show the proposed edit and apply it only after the user confirms. Don't commit the plan unless asked.
