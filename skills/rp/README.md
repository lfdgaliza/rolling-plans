# `/rp`: Rolling Plans skill for Claude Code

A Claude Code skill that knows the [Rolling Plans](../../README.md) conventions. You no longer have to point Claude at the methodology every time you want to mark a requirement completed or find the next one.

## Install

Clone this repo (`git clone https://github.com/lfdgaliza/rolling-plans`), then install the skill at the user level, so `/rp` works in every repo (plans and code usually live in different repos).

**Windows: link it (recommended).** With a junction, edits to the skill in this repo take effect right away. No admin rights needed:

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\rp" -Target "<path to this repo>\skills\rp"
```

**macOS / Linux: link it:**

```sh
ln -s "<path to this repo>/skills/rp" ~/.claude/skills/rp
```

**Or copy it.** Copy this folder to `~/.claude/skills/rp`. You'll need to copy it again after each change.

If `/rp` doesn't show up in a running session, start a new one.

## Usage

```
/rp [action] [plan name or path]
```

| Command | What happens |
|---|---|
| `/rp` | Status: completed requirements, the next one, the rest. |
| `/rp help` | Lists the actions and how to use them. |
| `/rp mark R1 completed PR #1201` | Reads the PR diff, rewrites R1 as-built, adds ✅ and the PR to the heading, and proposes a decision for anything that turned out differently. Asks for the PR if you don't give it. |
| `/rp next` | Names the next requirement and checks it against the code. Doesn't start implementing. |
| `/rp add rate-limit the PDF endpoint` | Adds a planned requirement at the end. |
| `/rp drop R3` | Deletes a planned requirement and records a decision saying why and where the work lives now. |
| `/rp check` | Reports where the plan and the code disagree, order problems, untracked work, stale notes. Doesn't fix them. |

Free text works too: `/rp mark r1 as completed` is the same as `/rp mark R1 completed`.

**Which plan.** Name it in the command (`/rp next externalCvrValidation`), or use one already in the conversation. Otherwise Claude looks for a `plans/` folder in the current directory and its parents, and asks if more than one plan fits.

**Nothing is saved without your OK.** Claude shows the proposed edit first and doesn't commit the plan unless you ask.

Claude can also load the skill on its own when you ask, in plain words, for something it covers, such as "R2 is done".
