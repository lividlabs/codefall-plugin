---
name: specify
description: Turn a feature idea into a specification another session can implement — interview for what the user will observe, push back on vague answers, audit what already exists, then write user stories with numbered acceptance criteria into the issue tracker.
argument-hint: "[what you want to build]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - AskUserQuestion
  - Write
  - Edit
  - Bash
  - WebSearch
  - WebFetch
---

# Specify

Turn a feature idea into a specification precise enough that another session can implement it
without re-interviewing anyone.

The output is **numbered acceptance criteria in an issue tracker**. Everything else this skill does —
the interview, the push-back, the audit of what already exists — is in service of making those
criteria correct.

Specifying is not designing. Once the criteria are written and confirmed, stop. How the thing gets
built is `design`'s work, and `implement`'s after that.

## Scope — what, not how

Specify decides **what will be true when this is done**. It does not decide **how it gets built**.

| In scope | Out of scope |
| --- | --- |
| Who the consumer is and what they get out of it | Which layer, component, or module the work lands in |
| What they can observe when it works | File paths, libraries, services, frameworks |
| What must keep working that already works | Schema, entity fields, API shapes, message contracts |
| What is explicitly not included | Work breakdown, ticket sequencing, dependency edges |
| Which questions remain unresolved | Estimates, effort sizing, build order |

This is a hard line, and crossing it is the most common way this skill fails. The tell is a
specification that names a technology. If the criteria mention a table, a class, an endpoint, or a
package, they have stopped describing the outcome and started describing the solution.

**You may read the codebase, but only to answer two questions**: does this already exist, and what
would this change silently break? Reading code to decide *how* to build the thing is `design`'s job
and is out of bounds here, even when the answer seems obvious.

Paths in this document are relative to `${CLAUDE_PLUGIN_ROOT}`. Resolve them against that root — they
are not relative to the user's project.

## What a specification is

One **user story** per issue, with its **acceptance criteria** underneath it.

### The user story

```
As a <consumer>, I want <capability>, so that <benefit>.
```

**The "so that" clause is mandatory.** A story without it names an action with no reason, and a
criterion written against it has nothing to be judged against. If the user cannot supply the benefit,
that is a signal the feature is not understood yet — ask, do not fill it in.

**The consumer is not always a person, and pretending otherwise makes the story worse.** A system can
be the consumer:

> As the BFF, I want to cache search results, so that a repeated search does not pay the upstream
> round trip again.

Reach for a human consumer first, because a human benefit is harder to fake. But do not manufacture
one. `As an engineer` is usually a punt — legitimate when building developer tooling, evasive
otherwise.

**When no real persona exists, say so in the story rather than inventing one:**

> A clean persona was hard to identify here; the closest consumer is the on-call engineer,
> because they are the only party who observes the outcome.

That sentence is more useful to the implementer than a plausible fiction.

### Acceptance criteria

Two kinds. Both are numbered, and both are written in the language of the domain.

**Scenarios** cover behavior:

```
AC-142-01
  Given a traveler with no saved trips
  When they open the trips list
  Then they see the empty state with a "Plan a trip" action
```

**Constraints** cover things that contort badly into Given/When/Then — data shapes, invariants,
inclusions:

```
AC-142-02
  The trip export includes the destination, departure date, return date, and traveler count.
```

Forcing a constraint into a scenario produces padded `Given a user exists` noise that reads as
ceremony. Use whichever kind states the fact plainly.

#### Criteria must be observable in a running system

Not observable in a test — observable in the product. This matters most for technical work, and it
has a consequence you must surface rather than absorb quietly.

Take the BFF cache above. "A repeated search does not call upstream" is only observable if something
emits that fact — a metric, a log line, a response header, an admin view. **That instrumentation is
now part of the requirement**, and it was arrived at by writing the criterion.

Say so out loud when it happens:

> Making this observable means the service has to expose cache hits somewhere. That is a real
> addition to the work. In scope, or do you want to leave it unverifiable for now?

Never quietly inflate the ticket, and never write a criterion nobody can check.

#### Implementation vocabulary is prohibited

`Given I POST to /api/trips` passes a syntax check and defeats the entire purpose. So does naming a
table, a component, a queue, or a library. A criterion describes what someone observes, in words they
would use.

The test: could this criterion still be true after a complete rewrite of the implementation? If not,
it is describing the solution.

#### Numbering

Criteria are identified as `AC-<issue-number>-<nn>`, zero-padded to two digits — `AC-142-01`,
`AC-142-07`. The issue number makes the identifier globally unique, so a test name or a `design`
ticket can cite `AC-142-07` with no ambiguity.

**This is why criteria cannot be written at issue-creation time** — the number does not exist until
the tracker assigns it. The creation sequence in each tracker profile handles this in two passes.

**Numbering is append-only.** A deleted criterion retires its number; the number is never reused. A
test citing `AC-142-07` must never silently come to mean a different criterion. This is the same
discipline this repo applies to ADR history.

#### Criteria exist for coverage, never for symmetry

**Never write a criterion to fill a section.** If a story's behavior is fully covered by four
criteria, write four. A parent issue with zero criteria of its own is a correct outcome when its
children cover everything. So is a parent that covers everything when the children are thin.

The only question is whether the behavior is covered. Balance across issues is not a goal, and a
template that looks lopsided is not a defect.

## Project customizations

If `.codefall/skills/specify/CUSTOMIZE.md` exists in the user's project, read it before step 1, say
you loaded it, and follow it for this run. It carries procedure this plugin cannot know — a system to
consult, a question this project always asks, a section every issue carries, a step that runs after
creating.

It extends this skill and never relaxes it: **Rules** below holds regardless. A customization that
would suspend one is asking for a different skill — say so and stop.

## Process

### 1. Ask what they want to build

One open question:

> "What would you like to build? A sentence or two is enough to start."

### 2. Check whether it already exists

Two searches, both cheap, both worth doing before spending the user's time on an interview.

**The codebase** — Glob and Grep for what they described. Filenames, exported identifiers, route
segments, domain nouns.

**The tracker** — the profile supplies the search command.

### 3. Report what you found

- **It exists in code.** Describe what is there, with file paths. Ask whether that is what they meant,
  and if not, what the distinction is.
- **An issue already covers it.** Link it. Ask whether it is the same thing.
- **Nothing found.** Say so and move on.

If the user confirms existing work covers their need, **stop the skill**. Nothing to specify.

### 4. Interview

This is the step that decides whether the specification is worth anything.

**Style.** One or two questions at a time, never a wall. Start broad and narrow. Use the user's own
vocabulary rather than translating it into jargon.

**Cover what is still unclear** — skip anything the description already settled:

1. Who the consumer is.
2. What they can do that they could not do before.
3. What starts it — an action, an event, a schedule.
4. What they observe when it works.
5. What happens when it does not — empty, unauthorized, upstream failure, bad input.
6. What this depends on that does not exist yet.
7. What is explicitly out of scope.

**Push back on vague answers.** Name the vague word and ask for a concrete replacement:

| They said | Ask |
| --- | --- |
| "fast" | Faster than what? What latency is acceptable, and at which percentile? |
| "good UX" | What does good look like here — a reference product, a specific interaction? |
| "manage" | Which actions: create, edit, delete, reorder, archive? |
| "integrate with X" | Which part of X — their search, their booking flow, their SSO? |
| "like before" | Like which screen, which flow? Walk me through it. |
| "just works" | What is the success path, and what is the failure path? |
| "real-time" | Under a second, or under a minute? |

Before advancing, judge the answers against consumer, trigger, observable outcome, and failure
behavior. **If fewer than roughly three of the applicable ones are concrete, do not advance.** Say
which ones are thin and ask.

**Insist for at most two rounds.** If the user overrules — "just write it with what we have" — write
it, and record every unresolved item under **Open questions** in the issue. Recorded, not resolved,
and never quietly dropped.

**Raise design concerns as flags, not rulings.** When the described experience has a problem the user
may not have considered, name it once and let them decide:

> "One thing I want to flag — a destructive action behind a hover has no reachable equivalent on
> touch. How are you thinking about that?"

Cap at two rounds per concern. If it stays unresolved, it goes under **Open questions** and you move
on. You are raising a concern, not overriding a decision, and the user knows their product.

**"Like $COMPANY does it."** When the user references another product, offer once to look it up. On
yes, research it and summarize only the patterns that matter — never dump page content. Confirm the
distillation with the user before it reaches the issue. On no, move on without searching.

### 5. Audit what already exists

A specification describes what the user wants **added**. If the surface already has functionality the
specification does not mention, an implementer may take the specification literally and remove it.

**Silent omission from a specification is not a deletion.** This is the rule that matters, and
authoring time is the only cheap moment to enforce it — by implementation time, nobody remembers the
specification was written against a stale picture.

For each surface the feature touches — a screen, a route, a component, an entity, a table:

1. **Read the current artifact.** Not its name, its contents. If it does not exist yet, skip; there is
   nothing to audit.
2. **Enumerate what is there.** Tabs, fields, states, columns, branches — whatever the unit of
   functionality is.
3. **Intersect with what the user described.** What did they account for? What did they leave out?
4. **If there is a real gap, ask.**

   > "The profile screen has three tabs today: Account Info, Travel Preferences, and Saved Passengers.
   > You mentioned the first two. Is Saved Passengers preserved as-is, folded into something new, or
   > intentionally going away?"

   Three outcomes:

   - **Preserved** — record it under **Existing behavior preserved** in the issue.
   - **Merged** — record how it composes, and cover it with a criterion.
   - **Removed** — this becomes its own acceptance criterion. **A removal is never an implicit
     consequence of the specification.**

Audit the surface the feature touches, not the whole application.

### 6. Decide whether to decompose

**Specify decomposes by what a user can observe. `design` decomposes by what can be built.** These
are different cuts, and conflating them puts build sequencing into a document that has no business
carrying it.

A child issue is a slice someone could see working on its own. A `design` ticket is work that can
land green — and one child may well become several of those.

**The sizing question**: could one person hold this whole thing in their head well enough to design it
in a single pass? If not, decompose.

**Depth is one level.** Parent and children, no grandchildren. When a child still looks too large,
say so and suggest a separate `specify` session for it.

**Push back once when the scope looks too big**, and carry a specific alternative — not "this is
large" but "I would cut it here, here, and here, because each of those is separately observable."

**Then defer.** If the user disagrees, write what they asked for. They know the domain, and what looks
like three capabilities from outside is often one trivial thing from inside. You may simply be wrong.

Record the concern in the parent issue **only** when the user did not engage with it — glossed over
it, or moved on without answering. Phrase it as an observation for `design` to weigh, not a warning.
If they considered it and disagreed, the matter is closed and nothing goes in the body.

**Criteria placement.** Children carry criteria local to their own slice. The parent carries only
criteria no single child could satisfy alone — the end-to-end journey, cross-step state, an invariant
spanning the whole flow. If a parent criterion could be checked by one child in isolation, it belongs
on that child.

**The parent closes when its children close.** They are one piece of work.

### 7. Mockups

When the feature has a visual surface, ask whether a mockup exists.

**If the user has one**, import it — follow `shared/import-mockup.md`. The imported files land under
`docs/mockups/<slug>/` and the issue references that path.

**If the user wants one but does not have it**, or has no design tool, the specification proceeds
without it and the issue is marked `requires-mockup`. `design` refuses to act on an issue carrying
that label, which is what keeps the gap from being forgotten.

**Do not draw a mockup inside this skill.** Producing one is separate work with its own concerns.

### 8. Recap before writing

The recap is the user's last chance to catch a misread, so it has to be substantive.

> Here is what I have. Tell me what is wrong.
>
> **Consumer**: …
> **What they can do**: …
> **What starts it**: …
> **What they observe**: …
> **Failure behavior**: …
> **Already exists and must keep working**: …
> **Out of scope**: …
> **Unresolved**: …
>
> Does that match what you have in mind?

**Every line is at least a sentence.** A line that collapses to a fragment or a dash means that topic
was not interviewed — go back and ask. The recap checks your work as much as the user's.

Advance when every line is substantive, the user has confirmed it, and you could write the criteria
without guessing at any of them.

### 9. Write, then confirm

Compose the full issue body and **show it to the user before anything is created**. Nothing reaches
the tracker unapproved.

```markdown
## User Story

As a <consumer>, I want <capability>, so that <benefit>.

## Context

[Two or three paragraphs in plain language: why this matters, what the user is doing when they
need it, how it fits what already exists. No technology.]

## Acceptance Criteria

[Filled in on the second pass, once the issue number exists.]

## Existing behavior preserved

[Only when step 5 found something. Otherwise omit the section entirely.]

- <element> — <what it does today>

## Out of scope

- <explicitly not included>

## Design notes

[Visual direction, mockup path, distilled research the user confirmed. Omit if empty.]

## Open questions

[Only when push-back did not resolve something. Otherwise omit.]

- <unresolved question>
```

**Omit empty sections.** A body full of headings with nothing under them trains readers to skim.

### 10. Create

Follow the creation sequence in the tracker profile. It runs in two passes because the criteria need
the issue number, and it is the profile's job to know how that works for its tracker.

### 11. Wrap up

Report what was created, with links. Ask whether anything needs adjusting.

## Tracker profiles

The specification is tracker-neutral. Where it lands, and in what shape, is a **tracker profile** —
one directory per tracker, exactly as `scaffold` handles surfaces.

| Tracker profile | Covers | Status |
| --- | --- | --- |
| `github` | GitHub Issues, optionally with a GitHub Project | **supported** |
| `jira` | Jira Cloud and Data Center | planned |
| `linear` | Linear | planned |

A tracker is **supported** only when `skills/specify/trackers/<name>/PROFILE.md` is complete. Nothing
else counts. A planned profile is an exit, not a menu choice — if the user's tracker is Jira, say
plainly that `specify` does not support it yet and stop.

**Resolution.** GitHub is currently the only supported profile, so there is no question to ask. State
that the specification will land in GitHub Issues and confirm the repository. When more profiles
exist, this step reads the project's recorded choice and asks only when there is none.

**Capabilities.** Each profile declares what its tracker can do — hierarchy, relations, custom fields,
markup, labels — and states a fallback for every capability it lacks. Read the profile's capability
table before writing, and follow its fallbacks rather than improvising around a missing feature.

## Rules

- **Nothing is created without the user confirming the full body first.**
- **The specification says what, never how.** No file paths, no libraries, no services, no schema.
  If a criterion would survive a rewrite, it is at the right altitude.
- **"so that" is mandatory** in every user story.
- **Criteria are observable in a running system**, and the instrumentation to make them so is part of
  the requirement — surfaced out loud, never absorbed silently.
- **Criteria exist for coverage, not symmetry.** Zero criteria on a parent is a valid outcome.
- **Numbering is append-only.** Retired numbers are never reused.
- **Silent omission is never a deletion.** A removal is always its own criterion.
- **Push back once, then defer.** On vagueness, on design concerns, on scope. The user knows the
  domain and you may be wrong.
- **Unresolved is recorded, not dropped.** Open questions go in the body.
- **Fall back gracefully** when tracker authentication is wrong: give the user the exact command to
  fix it and stop. Do not work around it.
