# Tracker profile: github

**Status:** supported.

GitHub Issues, in a single repository, optionally placed on a GitHub Project. This is the profile
`specify` uses when the specification's home is the same forge as the code.

## Fits when

- The project's issues live in a GitHub repository, **and**
- `gh` is installed and authenticated against that repository.

## Does not fit

A GitHub Project used **without** repository issues. Project draft items are cards that live only in
the project: they have no issue number, carry no repository labels, and cannot be parents or children.
Since acceptance-criteria identifiers are built from the issue number, a draft item cannot hold a
specification at all.

## Capabilities

| Capability | GitHub | What `specify` does |
| --- | --- | --- |
| Hierarchy | Native sub-issues | Parent/child via `addSubIssue` |
| Rich markup | GitHub-flavored Markdown | Body is written as plain Markdown, no translation |
| Labels | Repository-scoped, free-form | Two skill-owned labels; see below |
| Draft state | **Absent** | A `draft` label stands in |
| Custom fields | Only via a Project, and per-installation | Optional; see "Project boards" |
| Image attachment from CLI | **Absent** | Mockups are referenced by repository path |
| Dependency links | Present, unused | `design` owns dependency edges, not `specify` |

Two of those are absences worth stating plainly. **GitHub issues have no draft state** — the Projects
feature named "draft" is the unrelated card described above. And **`gh` cannot upload images**, which
is why mockups live in `docs/mockups/<slug>/` and the issue links to the path rather than embedding a
picture.

## Preconditions

`gh` must be authenticated with `repo` scope. When sub-issues or a project board are involved, also
`read:project` and `project`:

```
gh auth refresh -s read:project -s project
```

If authentication is wrong, give the user that command and stop. Do not work around it.

## Labels

This profile owns exactly two labels, and creates them if missing:

```bash
gh label create "draft" --color "BFBFBF" \
  --description "Specification incomplete — acceptance criteria pending" --force
gh label create "requires-mockup" --color "D93F0B" \
  --description "Blocked from design until a mockup exists" --force
```

Both are gates `design` reads. `draft` means the specification is not finished. `requires-mockup`
means the visual surface is unspecified. `design` refuses an issue carrying either.

**`specify` does not impose a label taxonomy.** Type, domain, and component labels are the project's
own business, and a curated set baked into this profile would be wrong for every project that did not
choose it. Apply labels the repository already uses when the fit is obvious; invent none.

## Field mapping

| Specification | GitHub |
| --- | --- |
| User story, context, criteria, scope, notes | Issue body, Markdown |
| Title | Issue title |
| Parent/child decomposition | Sub-issue relations |
| Specification incomplete | `draft` label |
| Mockup missing | `requires-mockup` label |
| Mockup present | Repository path in **Design notes** |

## Searching for duplicates

```bash
gh issue list --search "<keywords>" --state all \
  --json number,title,state,url,labels --limit 10
```

## Creating

Two passes, because acceptance criteria are identified as `AC-<issue-number>-<nn>` and the number does
not exist until GitHub assigns it.

**Pass one — create without criteria.**

```bash
gh issue create --title "<title>" --body "<body without the criteria section>" --label "draft"
```

Then record that the criteria are coming, so an issue caught mid-flight is not mistaken for a finished
one:

```bash
gh issue comment <number> --body "Acceptance criteria pending."
```

**Pass two — fill in the criteria.** Now that the number exists, number the criteria
`AC-<number>-01` upward, insert them under `## Acceptance Criteria`, and update the body:

```bash
gh issue edit <number> --body "<complete body>"
gh issue comment <number> --body "Acceptance criteria complete — <count> criteria."
gh issue edit <number> --remove-label "draft"
```

**`draft` comes off when the criteria land.** `requires-mockup` is independent and is cleared when a
mockup arrives, not here.

**When the specification decomposes**, create the parent first, then each child, then link them:

```bash
gh issue view <number> --json id --jq '.id'   # node ID for each issue
```

```bash
gh api graphql -f query='
  mutation {
    addSubIssue(input: { issueId: "<parent-node-id>", subIssueId: "<child-node-id>" }) {
      subIssue { number }
    }
  }'
```

Each child runs its own two-pass sequence — its criteria are numbered from its own issue number, not
the parent's.

**`specify` writes no dependency links between issues.** GitHub supports them; ordering work is
`design`'s decision, made against the build graph, and a guess recorded here would be a guess `design`
has to unpick.

## Project boards

Placing issues on a GitHub Project is **optional**, and this profile deliberately hardcodes nothing
about one.

Project IDs, field IDs, and single-select option IDs are **per-installation** values. They are not
properties of GitHub, and a profile that shipped them would work for exactly one repository. They have
to be discovered at run time:

```bash
gh project list --owner <owner>
gh project field-list <number> --owner <owner> --format json
```

Then place and set fields:

```bash
gh project item-add <number> --owner <owner> --url <issue-url>
gh project item-edit --project-id <project-node-id> --id <item-id> \
  --field-id <field-id> --single-select-option-id <option-id>
```

**This is the seam where per-project configuration belongs**, and it does not exist yet. Until it
does, ask the user whether issues go on a board, discover the IDs if they do, and skip the board
entirely if they do not. Never guess a field value, and never invent a workflow status the project
does not define.

## Note on Beads

`bd github sync` moves issue **content** — title, body, labels, state, assignee. It does not move the
graph. Reading `internal/github/mapping.go` at `main`, the GitHub-to-beads conversion returns an empty
dependency list unconditionally, and the beads-to-GitHub direction sends only title, body, labels, and
state. Sub-issue relations and dependency links are invisible to it in both directions.

So the parent/child structure written here does not reach Beads through sync. `design` reads hierarchy
from GitHub directly.
