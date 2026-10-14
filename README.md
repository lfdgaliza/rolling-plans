# Rolling Plans

*A plan that responds to change.*

> **Status:** Draft.

## What this is

Plans change. Halfway through a feature you learn something from a colleague, a code review, the docs or an AI session, and the plan you started with is already wrong. A rolling plan is built for that: one markdown file per feature that rolls forward with the work. It keeps you on track while things change, so you're never stuck following a plan that no longer fits.

How it works: requirements get added, removed and reshaped as you learn. When one is done, it is rewritten to describe what was actually built. The file always shows where things stand right now: what exists, what's next, and why.

The name nods to *rolling wave planning*, an established project-management technique: near-term work is planned in detail, later work is kept rough and refined as you get closer.

### Why use it

- **Low maintenance, more action.** One file per feature, a handful of conventions, no tooling. Compared with a spec-driven setup full of generated specs, there's very little to maintain, so most of your time goes into the work itself.
- **A tool for thought.** Writing the next requirement forces you to think it through before you build it. The goal, the open work and the reasoning stay in one place, so complex tasks don't get away from you.
- **Context for AI agents.** An agent reads the plan before touching code. Completed requirements tell it what already exists in the code. Planned ones tell it what to build next.
- **Adopt it alone.** It's a personal practice, not a team process. Nobody else has to agree to it or change how they work. The only things others see are better PRs and clearer decisions.

Every section can be written by hand, with AI help, or both. The plan's value comes from keeping it current, not from who typed it.

## Where plans live

```
plans/
  <agent instructions file>   ← conventions shared by all plans (code style, patterns)
  <Feature area>/
    <plan-name>.md            ← one rolling plan per piece of work
```

When work moves from one plan to another, add a decision in the old plan saying where it lives now, e.g. "Exports are handled in `<other-plan>.md` R2".

## Anatomy of a plan

### Core sections

Nearly every plan has these, in this order.

| Section | Purpose |
|---|---|
| Title | `# Plan: <feature> — <slice>`, so it's clear which part of a larger feature this plan covers. |
| Context | Why the work exists, what came before, and the current state. Date anything that will go stale ("as of 2026-05-05"). |
| Goal | What "done" means: the outcome, not how to get there, in a sentence or a few bullets. |
| Decisions | Constraints that apply across requirements, ideally with the reason for each. |
| Acceptance | How to tell the whole plan is done: builds, tests, end-to-end checks. |
| Requirements (R1…Rn) | The core of the plan: the units of work. See [Requirement lifecycle](#requirement-lifecycle). |

Requirements come last on purpose. The sections above them are short and change rarely, so they form a stable frame: whoever reads the plan top-down, person or agent, knows the constraints and what "done" means before reaching the work. Requirements are the long, moving part, and the timeline runs to the end of the file, so new work is added at the bottom.

### Optional sections

Add these when the feature needs them. They go before Requirements, with the rest of the frame.

| Section | Purpose |
|---|---|
| Scope | What is in and what is explicitly out. |
| Security considerations | Threats and mitigations specific to this feature. |
| Key files | Starting points in the codebase. See [Key files](#key-files). |
| Anything custom | Migration steps, rollback plan, open questions: whatever this feature needs. |

Rules that apply to the whole codebase (language version, formatting, which HTTP client to use) belong in the shared agent instructions file, not in each plan. Otherwise every plan repeats them and they eventually contradict each other.

## Requirement lifecycle

Each requirement is a `### Rn.` heading. A requirement is either completed or planned:

| State | Heading | How the body reads |
|---|---|---|
| Completed | `### ✅ R1. PDF generation endpoint (COMPLETED, PR #1201)` | **Rewritten as-built**, in past tense. Describes what exists in the code now. |
| Planned | `### R2. Download button — Frontend` | Future tense. Context, proposed implementation, acceptance. Code snippets are fine. |

**The order of the requirements is a timeline.** Completed requirements come first, then planned ones, always in that order. The next thing to work on is the first planned requirement, right after the last completed one. There's no separate "in progress" marker: if R1 is done, R2 is in progress. Completed requirements are history and don't move. Planned ones can be reordered, rewritten or removed as the plan changes. Removing one is recorded in [Decisions](#decisions).

### Rewriting as-built

A planned requirement describes intent:

```markdown
### R1. PDF generation endpoint — Backend

- Add `GET /invoices/{id}/pdf` that renders the invoice synchronously and returns the file
- Use the existing HTML invoice template as the source
```

When it is completed, the same requirement is rewritten to describe what exists in the code:

```markdown
### ✅ R1. PDF generation endpoint — Backend (COMPLETED, PR #1201)

- `POST /invoices/{id}/pdf` was added; it enqueues a render job and returns a job id
- `GET /invoices/pdf-jobs/{jobId}` returns the job status and, when ready, a download URL
- Rendering reuses the existing HTML invoice template
```

The plan said synchronous and the build turned out asynchronous. That change matters, so it becomes a decision:

```markdown
2. PDF generation is asynchronous, because invoices with hundreds of lines exceeded
   the request timeout
```

Guidelines:

- Past tense, factual: "`X` was added to `Y`", not "Add `X` to `Y`".
- If the result differs from the plan in a way that matters, record it as a [decision](#decisions) with the reason.
- Put the PR link in the heading. The PR is the detailed record, so older completed requirements can later shrink to a short summary.

### Anything that must happen is an R

Work that someone has to do must be a requirement, or a checkbox inside one. It must not sit only in prose sections like Security considerations or Acceptance. If a line there says "rate-limit the download endpoint" and has no status, nobody can tell whether it was done, dropped or forgotten.

## Decisions

The plan describes the current state, not how it got there. Git keeps the history. Anything important that changes ends up as a decision, stated as it stands now, with the reason when the reason isn't obvious:

```markdown
1. PDFs are generated server-side, never in the browser, because fonts rendered
   inconsistently across browsers
```

When a decision changes, rewrite it. Don't keep the old version struck through next to the new one.

Dropping a planned requirement also becomes a decision. Delete the requirement and state where the work lives now, if anywhere:

```markdown
3. Emailing the PDF is handled in `invoice-sharing.md`, which covers all outbound
   invoice delivery
```

## Tracking PRs

- A completed requirement has its PR number in the heading: `(COMPLETED, PR #1234)`.
- If one requirement spans several PRs or services, list them in a header block:
  ```markdown
  > **Service:** Backend + Frontend
  > **Branch:** feature/invoice-pdf
  > **PRs:** #1201 (backend), #1234 (frontend)
  ```
- A requirement isn't marked ✅ without a PR, unless it needed no code (e.g. manual configuration). In that case, say so.

## Notes and things to verify

Short inline notes are fine while a plan is moving. They record where a fact came from, or flag a claim nobody has checked yet:

```markdown
> Note: product owner says invoices over 50 pages are rare
> To verify: does the PDF library support right-to-left text?
```

They are temporary. Once a note is confirmed, fold it into the text, a requirement or a decision, then delete it. The goal is to keep claims honest, not to record who said what.

## Key files

Key file lists, especially AI-generated ones, go out of date quickly as code moves.

- Keep them **inside the requirement they belong to**. Don't keep a second plan-wide list, because the two drift apart.
- Treat them as hints for where to start looking. The code is the truth.
- Once a requirement is completed and has a PR link, its key files list can go. The PR diff is the exact list.

## How agents should use a plan

Put these rules in the agent instructions file:

1. If a plan is provided, read the whole thing before starting.
2. Treat ✅ requirements as facts about the current code, and planned ones as proposals to check against the code. The next requirement to implement is the first one without ✅.
3. If the code contradicts the plan, stop and say so instead of silently picking one.
4. After implementing a requirement, propose the as-built rewrite, plus a decision for anything important that turned out differently from the plan.

## Tips

- **Keep plans under version control.** The plan only shows the current state; git holds the history of what was intended and how it changed.
- **Shrink old requirements.** Once a requirement is completed and its PR is merged, cut it to 2–3 lines plus the PR link, so the plan stays short enough to read in full.
- **Draft section by section with AI.** Settle Context and Goal before asking for requirements. A plan generated in one pass looks complete but tends to blur the goal with how to get there.

## Example plan

A complete, small plan in the middle of implementation. R1 is done, so R2 is the one being worked on:

```markdown
# Plan: Invoice PDF Export — Download Flow

## Context

Users can view invoices in the app but can't download them; support gets ~30 requests
a week to email PDFs manually (as of 2026-05-05). The HTML invoice template already exists.

## Goal

- Users download any invoice as a PDF from the invoice page
- PDFs match the HTML invoice visually

## Decisions

1. PDFs are generated server-side, never in the browser, because fonts rendered
   inconsistently across browsers
2. PDF generation is asynchronous, because invoices with hundreds of lines exceeded
   the request timeout
3. Emailing the PDF is handled in `invoice-sharing.md`, which covers all outbound
   invoice delivery

## Acceptance

- Backend and frontend build; all tests pass
- End-to-end: open invoice → Download PDF → file matches the HTML invoice

## Requirements

### ✅ R1. PDF generation endpoint — Backend (COMPLETED, PR #1201)

- `POST /invoices/{id}/pdf` was added; it enqueues a render job and returns a job id
- `GET /invoices/pdf-jobs/{jobId}` returns status and, when ready, a download URL

### R2. Download button — Frontend

- Add a "Download PDF" button to the invoice page
- On click: start the job, poll the status endpoint, then trigger the download
- Show a spinner next to the label while the job runs; show an error message on failure
- Tests: happy path, failed job, polling timeout

> To verify: should polling stop after a fixed time or keep going until the job ends?

### R3. Rate-limit PDF generation — Backend

- Limit `POST /invoices/{id}/pdf` to 10 requests per user per minute
- Return 429 with a retry hint when the limit is hit
```
