---
name: reviewer-pr-body
description: Use whenever a pull request is opened, created, raised or updated — "open a PR", "create a PR", "raise a PR", "make a draft PR", "update the PR description", "push and PR", or before running `gh pr create` / `gh pr edit --body`. Builds a reviewer-first PR body from the ticket, its linked artefacts, the diff and the test evidence, so a reviewer can judge intent, risk and verification without opening the ticket or reading every file.
---

# Reviewer-First PR Body

A reviewer has three questions: *what should this do*, *what could go wrong*, *how do I know it works*. The diff answers none of them. The body does, or the reviewer reconstructs it from the ticket, Slack and the code — slowly, and differently from you.

Walk the steps in order, every time. Size comes first because it decides how much context is worth gathering: a config tweak does not need the ticket's Miro board.

## Step 1 — Classify the risk surface (from the diff)

Read `git diff <base>...HEAD` and every commit subject. Answer each yes/no:

1. **Trust**: does it accept a weaker guarantee than someone would assume? (auth, trust boundary, validation it cannot do)
2. **Authority**: does it change who can approve, access, deploy or sign anything?
3. **Data**: does the diff or ticket show existing records that stay wrong and need a manual fix? Can't tell → **no**, and it becomes a Notes question instead.
4. **Paths**: does more than one user-facing path change behaviour?
5. **Behaviour**: does the product behave differently for users or for the systems it talks to? CI, workflow, tooling, membership lists and docs count as **no**.
6. **Deploy**: does deploy order matter, or is there a migration/infra/config change?
7. **Scope**: does the diff include anything outside the ticket?
8. **Pending**: is any verification still not done?

## Step 2 — Pick the size

Size follows what the reviewer must decide, never the line count. A 20-line auth change needs more words than a 2,000-line rename.

```
Trust, Data or Paths = yes?
├── Yes → FULL
└── No
    ├── Behaviour = yes (includes most bug fixes) → STANDARD
    └── No (refactor, deps, copy, config, tests)  → SHORT
```

| Size | Word budget | Includes |
| --- | --- | --- |
| SHORT | ≤ 150 | TL;DR, ticket, changes, verify (≤ 2 rows), notes, checklist. No diagrams. |
| STANDARD | ≤ 350 | Adds a one-line **Cause** for bugs, one flowchart if a flow changed, verify ≤ 4 rows. |
| FULL | ≤ 800 | Every section in Step 4 that applies. Verify ≤ 8 rows. |

**Counting**: `wc -w` on the whole body minus mermaid blocks, code blocks and collapsed `<details>` blocks. Tables and headings count. The template's own fixed checklist and flag wording does not (your one-line flag reason does). Count it before publishing; never estimate. Why: reviewers judge length by what they scroll past, not by prose alone. A budget that exempts tables lets a body double in size while staying "within budget". Diagrams are exempt because they replace prose a reviewer would otherwise read.

Over budget → cut prose first, then move depth into `<details>`. Never cut a reviewer decision, a Notes bullet, or a verification row.

**Authority = yes at any size** → one bold line naming exactly who gains or loses what, ending "Confirm that is intended." It goes in the Reviewer decision section if there is one, otherwise in Notes. Why: permission changes look like config, so the size tree alone ranks them SHORT, yet they are the line a reviewer most needs to see.

## Step 3 — Gather context (scaled to size)

| Source | SHORT | STANDARD | FULL |
| --- | --- | --- | --- |
| Ticket key + link | if one exists | yes | yes |
| Ticket URL pattern (AGENTS.md, PR template, last 5 merged PR bodies) | if a key exists | yes | yes |
| Ticket body, acceptance criteria, comments | — | yes | yes |
| Linked artefacts (Miro, Figma, Confluence, specs) | — | if a flow changed | yes |
| Causing PR/commit (`git log -S`, `git blame`) | — | bugs | bugs |
| Failure evidence (failed CI run, error, log, alert) | bugs | bugs | bugs |
| Test evidence (this session's runs, `gh pr checks`) | yes | yes | yes |
| PR template (`.github/pull_request_template.md`) | yes | yes | yes |

### Finding the tracker

```
Does AGENTS.md / CLAUDE.md (root, then .github/) name a tracker?
├── Yes → use it (tool, key format, URL pattern, MCP/CLI to read it)
└── No
    ├── Branch name, commit subjects or PR title contain a key?
    │   ├── ABC-123       → Jira or Linear. Check which MCP is connected
    │   │                   (Atlassian → Jira, Linear → Linear). Both or neither → ask.
    │   │                   Atlassian MCP: call `getAccessibleAtlassianResources` first
    │   │                   for the cloud ID; never guess it from the org name.
    │   ├── #123 / gh-123 → GitHub issue (`gh issue view`)
    │   └── none          → check the PR template: "Link to Jira" etc. names the tool
    └── Still unknown → SHORT: write "None" and move on. Otherwise ask the user once.
```

When the tracker had to be inferred, end the run by proposing this note for the root `AGENTS.md` (show it; do not write it without approval):

```md
## Project tracking
- Tracker: Jira (your-org.atlassian.net), project key `PROJ`
- Ticket keys appear in branch names (`fix/proj-123`) and commit scopes (`fix(PROJ-123):`)
- Read tickets via the Atlassian MCP (`getJiraIssue`)
```

Why: inference costs a guess every run; one note in AGENTS.md makes every future run deterministic.

### When a source is unavailable

Ticket unreadable, artefact behind a login, no test counts: tell the user in chat what is missing and continue from the diff and tests. Never mention your own access, tools or limits in the body ("I could not read…", "from my session", "as far as I can tell"). Why: the body speaks for the author to the reviewer; the agent's plumbing is noise to them. If intended behaviour only the ticket can confirm is in doubt, ask the user before writing it.

## Step 4 — Write the body

Use the template's headings in order. Insert the extra sections at the marked positions.

**Template sections**: keep every heading that has content. Drop optional sections that would only say "None"/"N/A" (screenshots on a backend change, gifs). Always keep the ticket, checklist and feature-flag sections; "None" under Ticket tells the reviewer the omission is deliberate. Why: an empty heading costs the reviewer a scan and adds nothing.

**TL;DR** — 1–2 sentences of behaviour, including what deliberately does *not* change. Never name files or functions here. Fixes at any size, including SHORT, state the trigger (the error, failed run or request) in the TL;DR and link its evidence. Why: the motivation usually lives outside the diff, and without it the reviewer can't judge whether the fix is the right one.

**Ticket(s)** — link only. Build the URL from a pattern found in the repo (Step 3). No pattern found → the bare key; never guess a host.

**Cause** *(STANDARD bugs)* — 1–2 sentences after the ticket: what broke and why. Link the causing PR if one exists. Otherwise name what changed (in the code, data or world) in plain words. Never narrate the search.

**Explainer** *(FULL when Paths = yes; insert after Ticket. A FULL bug with Paths = no uses Why it broke instead)*
- One paragraph: the user's actions and why they diverge.
- Link the artefacts and state that the diagrams translate them.
- One mermaid flowchart per user-facing variant (e.g. per workflow type).
- One sentence on the scope edge: what is adjacent but out of this PR.

**Why it broke** *(FULL bugs)* — link the causing PR/commit, one paragraph of mechanism, then **Before** and **After** as two separate mermaid blocks, each under its own bold label line. Never put both in one block as subgraphs: Mermaid lays disconnected subgraphs side by side, so the diagram renders wide and scrolls. Name the code-level change in one sentence *after* the diagrams.

**Reviewer decision** *(each Trust = yes)* — heading contains "reviewer decision". State what the checks enforce, what they **do not** prove (bold), and the concrete misuse that remains. List alternatives only if the author actually weighed them (ticket, commits, conversation); otherwise omit that part. Never invent alternatives and never phrase residual risk as a guarantee. Why: a buried risk gets approved by accident; a named one gets accepted or rejected on purpose.

**What this cannot repair** *(Data = yes)* — what stays wrong and the manual action needed to fix it.

**Start here** *(FULL, or more than 5 files changed)* — one line: the file or function to open first and the order to read the rest. Why: on a large change, knowing where to start is the reviewer's hardest problem; on a small one the file list already answers it.

**Codebase changes** — one bullet per *intent*, not per file: SHORT 1–3, otherwise 3–6. Each bullet: the change and its reason in the same line. Out-of-scope changes get their own bullet ending "(not part of TICKET)"; they go in Notes only if you suggest splitting them out.

**Steps to verify**
- A `Scenario | Expected result` table. Each scenario is something a person does (clicks, commands, requests), and each result is what they see. For bugs, include what `main` shows instead, e.g. "On `main`: 'The connection is blocked…'". Why: the before-symptom proves the reviewer reproduced the right thing.
- Rows mirror the diagram branches, including negative paths (blocked, rejected, quarantined).
- Bugs: link the failure evidence (failed run, error, alert).
- Automated: what actually ran. From this session: suite + pass count. From CI only: the job names that cover this change, linked to the run. Never invent a count, and never claim coverage a job doesn't give ("none of these exercise the approval workflow" is a valid line). Nothing covers it → a Pending item in Notes.

**Follow-up sections** — every review round or out-of-scope fix goes in its own `<details><summary>Review round N: what changed</summary>` block below Steps to verify: what triggered it, what changed, how it was validated. Never fold them silently into Codebase changes: a re-reviewer needs to see only what is new.

**Notes to Reviewer** — one bullet per Step 1 "yes" that has no section of its own: Authority, Deploy, Scope, Pending. Never repeat a section ("see above"). Pending items are written as the PR's state ("Not yet verified on a live environment"), not the agent's. Never change draft/ready status yourself.

**Checklists / feature flags** — tick only what is true. Always write the one-line reason for the flag decision.

## Diagrams

- One flow per mermaid block, always `flowchart TD`, at most 3 branches from any decision. Why: the GitHub preview pane is narrow; anything wider scrolls sideways and stops being read.
- Nodes are actions and outcomes in plain language. Name components (services, queues, external systems) when the flow is between systems; never functions, files or routes. Why: the reviewer checks the diagram against the ticket, then the code against the diagram — two small checks instead of one big one.

## Prose that doesn't read as AI

Reviewers skim past text that sounds generated, and then they miss the one line that mattered. Write like the author's own commit messages.

- Delete any sentence that would be true of any PR ("This PR improves…", "ensures consistency", "In summary").
- Never use: comprehensive, robust, seamless, leverage, streamline, enhance, crucial, delve, ensure, "it's worth noting".
- Never explain what the diff already shows. Explain why, and what the diff can't show.
- One idea per sentence. Bold at most one phrase per section, and only for something a reviewer must not miss.
- No emoji, no "Overview"/"Summary" headings the template didn't ask for, no closing recap.
- Cut test: delete each sentence in turn. If the reviewer loses nothing, leave it deleted.

## Strict rules

- Every claim of behaviour traces to the ticket, an artefact, the diff or a test. No source → drop it.
- Every link is fetched this run or built from a URL pattern found in the repo. Never guess a host, URL or PR number.
- Update the body after every push that changes behaviour or scope. A stale body is worse than a short one: the reviewer trusts it.

## Before publishing

- [ ] Size picked from the Step 2 tree; `wc -w` count within budget
- [ ] Authority change named in a bold Notes line
- [ ] TL;DR readable by someone who has not opened the ticket
- [ ] Every verification row is an action a person can take; bugs show the `main` symptom
- [ ] No mention of the agent's own access or limits in the body
- [ ] No empty template sections, no repeated Notes
- [ ] Show the body to the user before `gh pr create`/`gh pr edit` — it is outward-facing
