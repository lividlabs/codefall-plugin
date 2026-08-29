---
name: mock-up
description: Get the visual surface of a feature into the repository under docs/mockups/ — import what a design tool exported, or make the mockup here when there isn't one, static or working depending on what the question is.
argument-hint: "[the screen or surface, or a path to a mockup you already have]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - AskUserQuestion
  - Write
  - Edit
  - Bash
---

# Mock-up

Show what a feature looks like, so that `design` and `implement` are not guessing at it.

Two ways in. Either the user has a mockup and this **imports** it, or they do not and this **makes**
one. Making one is the interesting half, and the one place in this plugin where inventing something
is the job rather than the failure.

The output is `docs/mockups/<slug>/` — the files, and a `README.md` saying what each one shows.

This skill runs before or after `specify`, and neither order is wrong. A mockup can be what makes the
requirements obvious, or it can be drawn once they are settled.

Paths in this document are relative to `${CLAUDE_PLUGIN_ROOT}`. Resolve them against that root — they
are not relative to the user's project.

## What it decides, and what it leaves alone

| This skill | `design` |
| --- | --- |
| What is on the screen, and its hierarchy | Which component or module renders it |
| The states the surface has | Where the data comes from |
| Realistic content at realistic lengths | Field names, types, schemas, endpoints |
| How an interaction behaves, when that is the open question | How that interaction is built in the app |

**Even a working mockup is a drawing.** One that runs proves the interaction; it does not become the
component. It was written to answer a question, not to the application's conventions, and it gets
rebuilt properly. That is the first rule in the directory's `AGENTS.md` and it holds for every file
this skill produces.

## The two modes

| The user has | Mode |
| --- | --- |
| Exported files, a directory of them, a design-tool URL, or an image pasted into the conversation | **Import** — follow `shared/import-mockup.md` |
| Nothing, or only part of what they need | **Make it** |
| A mockup already in `docs/mockups/<slug>/` that needs another state | **Amend** — add a file, never redraw the set |

A user who would rather draw it properly in their own tool should hear that running this again with
the export takes a minute. That is a remark, not a refusal — if they want something rough to look at
in the meantime, make it.

### Import

`shared/import-mockup.md` is the whole procedure, and it is shared with `specify` so the two never
drift. Follow it as written: what to accept, where the files land, the `README.md`, and the rules
about never editing or interpreting what the user brought. Then go to
[step 7](#7-clear-the-requires-mockup-label).

## Where mockups live

```
docs/mockups/<slug>/
```

`<slug>` names the **surface** — `booking-history`, `trip-share` — never the spec or the concept that
prompted it. One screen gets touched by several specs over its life and outlives all of them, so
filing it under whichever spec arrived first makes the second one either duplicate it or reach into
another spec's directory. `specify`'s SKILL.md carries the same rule for the same reason.

A run that covers several surfaces writes several directories. Reuse a surface's existing slug when
it has one; adding a state to a surface that has a directory is an amendment to that directory.

### The directory's rules file

`docs/mockups/AGENTS.md` is written when the directory is created, and added on a later run if it is
missing. `shared/import-mockup.md` writes the same file, so a directory reached from either verb
carries it.

**Never overwrite a file that has drifted.** When one exists and its content differs from what this
skill ships, show the user the difference and ask. The difference is a hand edit until they say
otherwise, and a repair that silently discards someone's rule is worse than a stale file.

```markdown
# docs/mockups — operative rules

- A mockup is a drawing of a surface. Never copy its markup, styles, or class names into the
  application — it is a picture of the outcome, not a draft of the implementation.
- A working mockup is still a drawing. It proves an interaction and gets rebuilt properly.
- Mockups are keyed by surface, never by spec or concept. One mockup serves several specs.
- Imported assets are the record of what someone decided. Never edit one; add alongside it.
- The `README.md` says what each file shows. Keep it current when files are added or replaced.
```

## Making one

Start from the defaults below and steer from there. They are where to begin, not what is permitted,
and every one of them gives way to a user who wants something else. Each carries the reason it is the
default, which is worth saying once when it looks like it might matter and never worth repeating.

| Default | Because | Go the other way when |
| --- | --- | --- |
| Static HTML, one file per state | A state behind a click is a state nobody reviews | The interaction itself is the open question |
| Self-contained — inline styles and script, nothing fetched | The file still opens years after the tool that made it is gone | A real library is what makes the mockup faithful |
| Greyscale plus one accent | A drawing that looks finished gets approved as finished | They want to see it finished — a pitch, a customer, a decision about the look |
| Realistic content, never lorem ipsum | Even-length placeholder text hides the wrapping that breaks layouts | Never, in practice |
| One stated viewport, drawn as a frame | It keeps breakpoint decisions from being made by accident | The reflow is the thing being decided |
| Screens as HTML | It opens anywhere and diffs in a pull request | The surface is not a screen — a terminal transcript, an email, a printed page |

**When the user asks for a fetched dependency, pin it.** A pinned CDN script is fine and sometimes
the only way to see the real thing; say once that the file then needs a network to open, and use the
version the project already depends on when it has one.

### Working mockups

When the question is how something behaves rather than what it looks like — a picker, a multi-step
flow, a drag interaction, a filter that has to feel right — build it working. That is a good use of
this skill and the reason the static default is a default.

Two things still hold. Every state a reviewer needs to see is reachable without reading the code,
which usually means the working file plus a still of each state that is hard to reach. And it is
still a reference: it proves the interaction, and the implementer rebuilds it in the app's stack.

### States worth having

A screen drawn only in its happy state is the most common way a feature comes back from review. Ask
which apply:

| State | Why it earns a file |
| --- | --- |
| Populated | The state everyone pictures already |
| Empty | First-run looks nothing like the populated screen, and nobody describes it |
| Loading | When it is slow enough to matter |
| Failed | What the user is told, and what they can do next |
| Denied | When the surface is permission-gated |
| Overflowing | Long content, many rows, a name that does not fit |

Populated, empty, and the primary failure are the ones to push for. If the user says a state does not
apply, take that and move on.

### The README

```markdown
# <Surface> mockups

Made <date> by `/mock-up`. <One line: what kind of thing these are — wireframes, a working
prototype, high-fidelity screens.> <Viewport, if it matters.>

- `<file>` — <one line: which screen or state this shows>

## Changes

- <date> — <what moved, and why>
```

Omit `Changes` until there is one. Keep the per-file lines current: an implementer opening a
directory of files cannot tell the empty state from the error state, and the person who could tell
them is no longer in the conversation.

**A directory can hold both imported and authored files.** A user who exported two screens and wants
the failure state drawn gets exactly that, and the README says which is which. **Never redraw an
imported asset** — draw the missing state alongside it.

## Project customizations

Follow `shared/customizations.md` for this verb.

## Process

### 1. Ask what this is for

**No precondition check.** This skill needs a place to write and nothing else.

Ask which spec this is for, and read `docs/specs/` if it exists. A spec answers most of what an
interview would: it names the surface, who the consumer is, and what they observe when it works.
Requirements marked `requires-mockup` are the ones waiting on this run.

**No spec, then look for a concept.** Read the live concepts in `docs/concepts/` if the directory
exists, and offer the relevant one:

> CONCEPT-002 covers the auditing rework. Want me to work from it?

A concept is a wider frame than a spec and usually names several surfaces, so a run from one can be
large. **Say the size out loud, then do what they ask:**

> That concept touches four surfaces. Drawing all of them is a dozen files or so. All four, or start
> with the one you most want to see?

**Neither is fine.** Plenty of surfaces get drawn before anyone writes anything down, and that is one
of the reasons this skill exists. Ask what they want to see and go.

Read `docs/mockups/` either way, for the slugs already in play.

### 2. Name the surface

Settle the slug before anything is written — the surface, not the spec or the feature request. Reuse
an existing one where the surface already has a directory.

**Do not ask where the files go.** `docs/mockups/<slug>/` is the answer.

### 3. Pick the mode

Ask what they have and route with [the modes table](#the-two-modes).

### 4. Fill the gaps

Only what a spec, a concept, or the user's own description did not already answer:

1. Which screen, and who is looking at it.
2. What is on it, and the one thing someone comes here to do.
3. Which states apply — walk [the states table](#states-worth-having).
4. Whether anything needs to actually work.
5. How finished it should look, if they have a view.
6. Anything in the product it should resemble, and where that lives.

**Two rounds is a cap on your insistence, not on the conversation.** Stop pressing an unanswered
point after the second try, make the sensible choice, and say in the recap that you assumed it.

**Do not bikeshed.** Which of two labels reads better is not worth a question — make one and let the
user move it.

### 5. Make one thing, then stop

Build the first screen or component and show it: the path, and what to open it with.

This is the step that matters most. Twelve files in a house style the user dislikes is twelve files
of rework, and the corrections that arrive on the first one — density, ordering, vocabulary, how
finished it should look — apply to all of them.

Advance when they have actually looked at it.

### 6. Make the rest, and write the directory

Apply every correction from the first one. Then write `README.md`, and `docs/mockups/AGENTS.md` if it
was missing. Report each file with what it shows.

Do not commit.

### 7. Clear the `requires-mockup` label

Follow the **Hand back** section of `shared/import-mockup.md` — it works the same for a mockup made
here as for an imported one. `specify` applies the label per requirement, so clear the issues this
mockup covers, leave the rest, and clear none of them unless files actually landed.

### 8. Link it back

When a spec prompted this run, offer to add the directory path to its **Design notes**. When a
concept did, offer to add it to the concept's `Related` line. Add the path and change nothing else in
either document — they belong to `specify` and `conceptualize`.

### 9. Wrap up

Report the directories, every file with what it shows, states deliberately not drawn and why,
anything assumed because the user did not answer, and any labels cleared.

Do not commit. Do not create issues. Do not start a design.

## Rules

- **Show the first one before making the rest.**
- **Never edit an imported asset.** It is the record of what someone decided. Add alongside it.
- **A mockup is a reference, never source.** Working ones included.
- **Mockups are keyed by surface**, never filed under a spec or a concept.
- **Say what it looks like, not how it is built.** Naming a component, a route, or a data source is
  `design`'s work happening in the wrong document.
- **Clear `requires-mockup` only when files landed**, and only on the issues this mockup covers.
- **Nothing lands outside `docs/mockups/`.** No application code, no styles, no components.
