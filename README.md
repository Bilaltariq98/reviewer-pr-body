# reviewer-pr-body

[View on skills.sh](https://skills.sh/bilaltariq98/reviewer-pr-body/reviewer-pr-body)

An agent skill for writing pull request descriptions that answer a reviewer's three questions: what should this do, what could go wrong, and how do I know it works.

It helps an agent:

- find the project tracker (Jira, Linear, GitHub Issues) from `AGENTS.md`, branch names, commits or the PR template, and read the ticket plus its linked artefacts
- classify the risk surface before writing: trust boundaries, unrepaired data, deploy order, scope creep, pending verification
- turn the ticket's scenarios into behaviour flowcharts, with before/after diagrams for bugs
- call out residual risk as an explicit reviewer decision instead of burying it
- write a verification table whose rows mirror the diagram branches, with real test counts
- keep the repo's own PR template headings and append a section per review round

## Install

Install with the [Skills CLI](https://skills.sh):

```bash
npx skills add Bilaltariq98/reviewer-pr-body
```

Install globally:

```bash
npx skills add Bilaltariq98/reviewer-pr-body --global
```

You can also inspect the skill before installing:

```bash
npx skills add Bilaltariq98/reviewer-pr-body --list
```

## Tracker discovery

The skill reads tickets through whatever your agent has connected: an Atlassian or Linear MCP server, or the `gh` CLI for GitHub Issues. To skip inference, add a note to your root `AGENTS.md`:

```md
## Project tracking
- Tracker: Jira (your-org.atlassian.net), project key `PROJ`
- Ticket keys appear in branch names (`fix/proj-123`) and commit scopes (`fix(PROJ-123):`)
- Read tickets via the Atlassian MCP (`getJiraIssue`)
```

When the tracker has to be inferred, the skill proposes this note at the end of the run.

## Usage

The skill activates when you ask the agent to open, create, raise or update a pull request, or before it runs `gh pr create` / `gh pr edit --body`.

Examples:

- "Open a draft PR for this."
- "Push and PR."
- "Update the PR description with the review fixes."

See [`SKILL.md`](./skills/reviewer-pr-body/SKILL.md) for the full instructions.

## License

[MIT](./LICENSE)
