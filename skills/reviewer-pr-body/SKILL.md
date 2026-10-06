---
name: reviewer-pr-body
description: Use whenever a pull request is opened, created, raised or updated — "open a PR", "create a PR", "raise a PR", "make a draft PR", "update the PR description", "push and PR", or before running `gh pr create` / `gh pr edit --body`. Builds a reviewer-first PR body from the ticket, its linked artefacts, the diff and the test evidence, so a reviewer can judge intent, risk and verification without opening the ticket or reading every file.
---

# Reviewer-First PR Body

A reviewer has three questions: *what should this do*, *what could go wrong*, *how do I know it works*. The diff answers none of them. The body does, or the reviewer reconstructs it from the ticket, Slack and the code — slowly, and differently from you.

Walk the steps in order, every time. Do not start writing until Step 3 is done: a body written before the context is gathered describes the code, not the behaviour.

## Step 1 — Find the project tracker

```
Does AGENTS.md / CLAUDE.md (root, then .github/) name a tracker?
├── Yes → use it (tool, key format, URL pattern, MCP/CLI to read it)
└── No
    ├── Branch name, commit subjects or PR title contain a key?
    │   ├── ABC-123      → Jira or Linear. Check which MCP is connected
    │   │                  (Atlassian → Jira, Linear → Linear). Both or neither → ask.
    │   ├── #123 / gh-123 → GitHub issue (`gh issue view`)
    │   └── none          → check the PR template: "Link to Jira" etc. names the tool
    └── Still unknown → ask the user once: "Which tracker and ticket?"
```

When the tracker had to be inferred, end the run by proposing this note for the root `AGENTS.md` (show it; do not write it without approval):

```md
## Project tracking
- Tracker: Jira (your-org.atlassian.net), project key `PROJ`
- Ticket keys appear in branch names (`fix/proj-123`) and commit scopes (`fix(PROJ-123):`)
- Read tickets via the Atlassian MCP (`getJiraIssue`)
```

Why: inference costs a guess every run; one line in AGENTS.md makes every future run deterministic.

## Step 2 — Gather context

Collect all of these before writing. Missing one → say so in the body, never paper over it.

1. **Ticket**: summary, acceptance criteria, comments. Comments often hold the decision that changed scope.
2. **Linked artefacts** in the ticket (Miro, Figma, Confluence, Loom, specs). Open them. These are the source of truth for intended behaviour, and a reviewer trusts a diagram traced to them over one you invented.
3. **The causing change** (bugs only): `git log -S`/`git blame` the broken path to the PR or commit that introduced it.
4. **The full diff** against the base branch: `git diff <base>...HEAD`, all commits, not just the latest.
5. **Test evidence**: which suites ran, pass counts, what is still unverified (live envs, manual checks).
6. **The repo's PR template** (`.github/pull_request_template.md`). Fill it; never replace it — reviewers scan for its headings.

## Step 3 — Classify the risk surface

Answer each with yes/no. Every "yes" becomes a mandatory section in Step 4.

- Does it accept a weaker guarantee than someone would assume? (auth, trust boundary, validation it cannot do)
- Does it fail to fix existing bad data/history?
- Does deploy order matter, or is there a migration/infra/config change?
- Does the diff include anything outside the ticket's scope?
- Is any verification still pending?

## Step 4 — Write the body

Use the template's headings in order. Insert the extra sections at the marked positions.

**TL;DR** — 1–2 sentences of user-visible behaviour, including what deliberately does *not* change. Never name files or functions here: the reviewer has not earned that context yet.

**Ticket(s)** — link only.

**Explainer** *(insert after Ticket; required when behaviour has more than one path)*
- One paragraph: the user's actions and why they diverge.
- Link the artefacts from Step 2.2 and state that the diagrams translate them.
- One mermaid `flowchart TD` per user-facing variant (e.g. per workflow type). Nodes are user actions and outcomes in plain language, never function names. Why: the reviewer checks the diagram against the ticket, then the code against the diagram — two small checks instead of one big one.
- State the scope edge in one sentence: what is adjacent but out of this PR.

**Why it broke** *(bugs only)* — link the causing PR/commit, one paragraph of mechanism, then a **Before** and **After** flowchart of the same flow. Name the code-level change (route, event, command) in one sentence *after* the diagrams.

**Reviewer decision** *(required for each "yes" on trust/guarantee in Step 3)* — heading must contain "reviewer decision". State: what the checks enforce, what they **do not** prove (bold), the concrete misuse that remains, which alternatives were considered and why each was rejected. Never phrase residual risk as a guarantee. Why: a buried risk gets approved by accident; a named one gets accepted or rejected on purpose.

**What this cannot repair** *(required when existing data is affected)* — what happens to history/old records, and the manual action needed to fix them.

**Codebase changes** — 3–6 bullets, one per *intent*, not per file. Each bullet: the change + the reason in the same line.

**Steps to verify**
- A `Scenario | Expected result` table. Rows mirror the explainer diagrams' branches, including the negative paths (blocked, rejected, quarantined). Why: the reviewer can tick the diagram off row by row.
- Then automated verification: suite name + pass count, plus lint/type/clippy checks. Never write "tests pass" without a number.

**Follow-up sections** — every review round or out-of-scope fix gets its own `##` section appended below Steps to verify: what triggered it, what changed, how it was validated. Never fold them silently into Codebase changes: a re-reviewer needs to see only what is new.

**Notes to Reviewer** — bullets, one per Step 3 "yes": deploy order, infra/config steps, unrepaired data, pending verification. If verification is pending, say the PR stays in draft until it is done.

**Checklists / feature flags** — tick only what is true. Leave unchecked items unchecked. Always write the one-line reason for the flag decision.

## Strict rules

- Every claim of behaviour traces to the ticket, an artefact, or a test. No source → drop the claim or mark it as an assumption.
- Every link is real and fetched this run. Never guess a URL or PR number.
- Numbers over adjectives: "37 tests passed", not "well tested".
- Diagrams describe behaviour; prose names code. Never put identifiers in diagram nodes.
- Update the body after every push that changes behaviour or scope. A stale body is worse than a short one: the reviewer trusts it.

## Before publishing

- [ ] TL;DR readable by someone who has not opened the ticket
- [ ] Each diagram branch has a matching verification row
- [ ] Each Step 3 "yes" has its section or Notes bullet
- [ ] Pending verification stated; draft status matches
- [ ] Show the body to the user before `gh pr create`/`gh pr edit` — it is outward-facing
