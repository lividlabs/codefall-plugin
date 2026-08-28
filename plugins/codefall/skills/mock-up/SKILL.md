---
name: mock-up
description: Get the visual surface of a feature into the repository — import a mockup exported from a design tool, or author one as self-contained static HTML when there is no design tool — landing it under docs/mockups/ keyed by surface, and clearing the requires-mockup label that blocks design.
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

The output is a directory at `docs/mockups/<slug>/` holding one file per screen state and a
`README.md` that says what each one shows. The files get there one of two ways: **imported** from a
design tool the user already works in, or **authored** here as static HTML when there is no design
tool.

This skill runs before or after `specify`, and neither order is wrong. A mockup can be what makes the
requirements obvious, or it can be drawn once the requirements are settled.

Mocking up is not designing. It says what the surface looks like and what states it has. It does not
say which components render it, where the data comes from, or how the work is broken up.

Paths in this document are relative to `${CLAUDE_PLUGIN_ROOT}`. Resolve them against that root — they
are not relative to the user's project.

## Scope — what it looks like, not how it is built

| In scope | Out of scope |
| --- | --- |
| What is on the screen, and its hierarchy | Which component, layer, or module renders it |
| The states the surface has — populated, empty, failing | Where the data comes from |
| Realistic content at realistic lengths | Field names, types, schemas, endpoints |
| Which screen the user reaches this one from | Routing, navigation implementation, state management |
| The reading order and what the eye lands on first | Brand color, illustration, iconography |
| Which viewport it is drawn at | Responsive behavior between viewports |

The tell that this skill has failed is a mockup somebody copies into the application. What lands here
is a **drawing of a surface**, not a first draft of the implementation. It is written to be looked at
and thrown away.

## The two modes

Ask what the user has before doing anything else. The answer decides the whole run.

| The user has | Mode |
| --- | --- |
| Exported files, a directory of them, a design-tool URL, or an image pasted into the conversation | **Import** — follow `shared/import-mockup.md` |
| Nothing, and no design tool they want to use | **Author** — draw it here |
| Nothing, but a design tool they intend to open themselves | **Neither.** Say so and stop |
| A mockup already in `docs/mockups/<slug>/` that needs another state | **Amend** — see [Amending an existing mockup](#amending-an-existing-mockup) |

**The third row is a real answer, not a failure.** A user who is about to draw the thing properly
does not want a placeholder in the repository competing with it. Leave the directory alone, leave any
`requires-mockup` label in place, and say that running this skill again with the export takes a
minute.

### Import

`shared/import-mockup.md` is the whole procedure, and it is shared with `specify` so the two never
drift. Follow it as written. It covers what to accept, where the files land, the `README.md`, and the
rules about never editing or interpreting what the user brought.

Import is not this skill's interesting half. Read the file, do what it says, and go to
[step 7](#7-clear-the-requires-mockup-label).

### Author

The rest of this document is the authoring mode.

## Where mockups live

```
docs/mockups/<slug>/
```

`<slug>` is a short kebab-case identifier for the **surface** — `booking-history`, `trip-share` —
never for the spec or the concept that prompted it. One screen gets touched by several specs over its
life and outlives all of them, so filing it under the spec that happened to arrive first makes the
second spec either duplicate it or reach into another spec's directory. `specify`'s SKILL.md carries
the same rule for the same reason.

Reuse the surface's existing slug when one is already in play. Adding a state to a surface that has a
directory is an amendment to that directory, not a new one.

### The directory's rules file

`docs/mockups/AGENTS.md` is written when the directory is created, and added on a later run if it is
missing. `shared/import-mockup.md` writes the same file, so a mockup that arrived through `specify`
gets it too.

**Never overwrite a file that has drifted.** When one exists and its content differs from what this
skill ships, show the user the difference and ask. The difference is a hand edit until they say
otherwise, and a repair that silently discards someone's rule is worse than a stale file. Replace it
only on a yes; on a no, leave it and say nothing further about it.

The file's content:

```markdown
# docs/mockups — operative rules

- A mockup is a drawing of a surface. Never copy its markup, styles, or class names into the
  application — it is a picture of the outcome, not a draft of the implementation.
- Mockups are keyed by surface, never by spec or concept. One mockup serves several specs.
- Imported assets are the record of what someone decided. Never edit one; add alongside it.
- The `README.md` says what each file shows. Keep it current when files are added or replaced.
```

## What an authored mockup is

**One self-contained HTML file per screen state.** No build step, no dependencies, no assets.

```
docs/mockups/booking-history/
  README.md
  01-list-populated.html
  02-list-empty.html
  03-list-load-failed.html
  04-detail.html
```

The number prefix is reading order, not a permanent identifier — it exists so the directory listing
tells someone where to start. The rest of the name says which screen and which state.

### House rules for the markup

- **Self-contained.** One `<style>` block in the file, a system font stack, and nothing fetched:
  no external stylesheet, no web font, no image URL, no CDN. A mockup that fetches is a mockup that
  breaks the first time someone opens it on a plane, and these files get read years after the tool
  that would have re-rendered them is gone. Use inline SVG or a CSS shape where a picture is needed,
  and a labelled grey box where the picture is beside the point.
- **No JavaScript, and no interactivity.** Every state is its own file. A state hidden behind a click
  is a state nobody reviews, and the states are the deliverable.
- **Greyscale, plus one accent color at most.** Color is a decision this skill was not asked to make,
  and a mockup that arrives fully styled gets approved as a brand direction by accident.
- **Realistic content, never lorem ipsum.** Real-looking names, real-looking dates, a genuinely long
  string that has to wrap, a value that is missing. A layout that only holds together with
  even-length placeholder text has not been tested, and the wrapping case is exactly what the
  implementer needs to see.
- **One stated viewport per file, and no media queries.** Put the width in a comment at the top and
  draw the frame at it. When both a desktop and a phone layout matter, they are two files, because a
  mockup that reshapes as the window is dragged has smuggled in breakpoint decisions nobody made.
- **Label the frame from outside it.** The `<title>` is `<Surface> — <state>`, and a line of small
  grey text sits above the frame saying the same thing. Nothing explanatory goes inside the frame,
  where it reads as part of the interface.

### Fidelity is deliberately low

Structure, hierarchy, content, and states. Not brand color, not illustration, not iconography, not
spacing worked out to the pixel.

The reason is consent. A drawing that looks finished gets treated as finished, and every visual
decision in it was made by a model nobody asked to make visual decisions. A grey wireframe invites
the correction it needs; a polished screen invites a nod.

**One exception, and only when the user points at it:** if the project already has screens, a
component library, or a design system in the repository, offer once to read them and match their
vocabulary — the same controls, the same layout conventions, the same names for things. That is
matching what exists rather than inventing, and it makes the mockup easier to read. Do not go looking
for it unprompted, and do not go looking at other products on the web.

### States are the point

A screen drawn only in its happy state is the most common way a feature comes back from review. Ask
which of these apply and draw every one the user confirms:

| State | Why it earns a file |
| --- | --- |
| Populated | The state everyone pictures already |
| Empty | First-run and nothing-yet look nothing like the populated screen, and nobody describes them |
| Loading | Only when it is slow enough to matter — say so rather than drawing it reflexively |
| Failed | What the user is told and what they can do next |
| Denied | When the surface is permission-gated at all |
| Overflowing | Long content, many rows, a name that does not fit |

At minimum: populated, empty, and the primary failure. If the user says a state does not apply, take
that and move on — the second row of the table is worth pressing on once and no more.

### The README

Every authored directory carries one. It parallels the imported README in
`shared/import-mockup.md`, and says plainly that these were drawn rather than exported:

```markdown
# <Surface> mockups

Authored <date> by `/mock-up`. Low-fidelity wireframes — structure and states, not visual design.
Drawn at <width>px.

- `<file>` — <one line: which screen or state this shows>
- `<file>` — <one line>

## Changes

- <date> — <what moved, and why>
```

Omit the `Changes` heading until there is a change to record. Keep the per-file lines current; an
implementer opening a directory of files cannot tell the empty state from the error state, and the
person who could tell them is no longer in the conversation.

**A directory can hold both imported and authored files.** A user who exported two screens and wants
the failure state drawn gets exactly that, and the README says which is which. The line from
`import-mockup.md` still holds: **never redraw an imported asset.** Draw the missing state alongside
it.

## Amending an existing mockup

Adding a state to a surface that already has a directory is the normal case, and it adds a file. It
does not redraw the set.

Replacing a screen is different, because a spec may already reference the picture that is being
replaced. Ask before overwriting, say what changed in the README's `Changes` list, and keep the
filename stable if the file is cited anywhere. Imported assets are never touched either way.

## Project customizations

Follow `shared/customizations.md` for this verb.

## Process

### 1. Look around

**No precondition check.** This skill needs a place to write and nothing else.

Read `docs/mockups/` if it exists — the slugs already in play, and the READMEs that say what they
hold. If the surface the user is describing already has a directory, say so before going further;
this is probably an amendment.

Then read `docs/specs/` if it exists, for a spec whose **Design notes** reference a mockup path, or
whose requirements are waiting on one. Offer the relevant spec as context:

> SPEC-004 covers the booking history screen and its requirements are marked `requires-mockup`. Want
> me to work from it?

A spec is **never required**. Plenty of surfaces get drawn before anyone writes requirements, and
that order is one of the reasons this skill exists.

### 2. Name the surface

Settle the slug before anything is written. It names the surface — `booking-history` — not the spec,
the concept, or the feature request. Reuse an existing slug when the surface already has one.

**Do not ask where the files go.** `docs/mockups/<slug>/` is the answer, and it is not a question.

### 3. Pick the mode

Ask what they have, and route with [the modes table](#the-two-modes). On import, follow
`shared/import-mockup.md` and go to [step 7](#7-clear-the-requires-mockup-label).

### 4. Interview

Only for what is genuinely missing. Skip anything a spec or a concept already answered.

1. Which screen, and who is looking at it.
2. What is on it, in their words, and the one thing someone comes here to do.
3. Which states apply — walk [the states table](#states-are-the-point).
4. Which viewport, or both.
5. Anything in the product it should resemble, and where that lives in the repository.
6. Anything that must not appear on it.

**Two rounds is a cap on your insistence, not on the conversation.** Stop pressing an unanswered
point after the second try, draw the sensible thing, and say in the recap that you assumed it. If the
user is still describing the screen, keep going with them.

**Do not bikeshed.** Which of two labels reads better, and where a control sits when either is fine,
are not this document's problems. A mockup exists to be corrected — draw one and let the user move
things.

### 5. Draw one screen, then confirm

**Write the populated state first, and stop.** Show the user the file path, tell them to open it, and
ask what is wrong with it.

This is the most important step in the skill. Six screens drawn in a house style the user dislikes is
six screens of rework, and the corrections that arrive on the first one — the density, the ordering,
the vocabulary — apply to all of them.

Advance when the user has actually looked at it and said it is close enough to continue from.

### 6. Draw the rest, and write the directory

Draw the remaining states, applying every correction from the first screen. Then write `README.md`,
and `docs/mockups/AGENTS.md` if it was missing.

Report each file with its one-line description before moving on, so a state drawn from a misreading
gets caught here rather than in `design`.

Do not commit.

### 7. Clear the `requires-mockup` label

Clearing works the same for an authored mockup as for an imported one — follow the **Hand back**
section of `shared/import-mockup.md`. `specify` applies the label per requirement, so a feature can
have several issues carrying it: clear the ones this mockup covers, leave the rest, and clear none of
them unless files actually landed.

### 8. Link it from the spec

When a spec prompted this run, offer to add the directory path to its **Design notes**:

> SPEC-004's Design notes are empty. Want me to add `docs/mockups/booking-history/` to them?

On a yes, add the path and change nothing else in the document. The spec belongs to `specify`, and
this is a reference being completed, not a spec being edited.

### 9. Wrap up

Report the directory path, every file with what it shows, which states were deliberately not drawn
and why, anything you assumed because the user did not answer, and the labels that were cleared.

Do not commit. Do not create issues. Do not start a design.

## Rules

- **Nothing is written without the user confirming the first screen.** The rest follow from it.
- **The mockup says what it looks like, never how it is built.** Naming a component, a route, or a
  data source is `design`'s work happening in the wrong document.
- **Never edit an imported asset.** It is the record of what the user decided. Draw alongside it.
- **Self-contained files only** — no fetch, no dependency, no build step, no JavaScript.
- **Low fidelity by default.** Match an existing design system only when the user points at one.
- **Every state is its own file.** A state behind a click is a state nobody reviews.
- **Realistic content.** Placeholder text hides the wrapping cases that break layouts.
- **Mockups are keyed by surface**, never filed under a spec or a concept.
- **Clear `requires-mockup` only when files landed**, and only on the issues this mockup covers.
- **Nothing lands outside `docs/mockups/`.** This skill writes no application code, no styles, and no
  components.
