# Roadmap

The objective of this plugin is to create skills to streamline the operations of
a software factory with an opinionated approach. Future versions will allow for
customization but the first version will be a fixed approach.

## Process via Skills

The process resembles the waterfall process of yore but the ability it iterate on functionality end-toend within hours instead of months.

 - **conceptualize** an idea into a short document that says what the problem is and why it matters, before anyone specifies or scaffolds it; concepts live in the repo at `docs/concepts/CONCEPT-NNN-slug.md`. `scaffold` requires one — a requirement the user can break, recorded in the decision log when they do — because its worst failure is picking architecture defaults from a one-sentence description. `specify` may draw on one and never requires it.
 - **scaffold** the architecture and this process
 - **specify** requirements for a feature and acceptance test criteria; specifications live inside of an issue tracker, like Jira, GitHub Issues, or Linear. The first version will only suport Github Issues
 - **mock-up** the visual surface of a feature; runs before or after specify. It imports mockups exported from a design tool, or helps create one when the user doesn't have or want a design tool. Mockups land in `docs/mockups/<slug>/` and the specification references them; an issue labelled `requires-mockup` blocks design until the mockup exists.
 - **design** a system to implement a feature, the output will a technical specification and a work breakdown; the tickets from this step will live inside of beads; this step also determines dependencies on the tickets it creates as well as any potential dependencies on existing ticket. It inserts the work into the graph.
 - **implement** a feature following the dependency graph and executing work in parallel waves where possible; unit, integration, and end-to-end tests are part of the defniition of done. It also owns the concept transition to `Active` — a concept is Active once work has started against it, and no earlier skill can observe that moment.
 - **review** code and tests with or without another model or harness
 - **test** the completed code with unit tests, integration tests, end-to-end tests, and agentic driven tests

 <insert a diagram?>

 ## Roadmap

 - [x] scaffold skill for Typescript projects: Node.js, React, and/or Next.js
 - [ ] conceptualize skill: concept documents in `docs/concepts/`, required by scaffold, optional for specify
 - [ ] scaffold skill for Golang projects
 - [ ] specify skill with Github Issues integration
 - [ ] mock-up skill: import from a design tool, or author one when there isn't one
 - [ ] specify tracker profiles for Jira and Linear
 - [ ] tracker resolution for specify: read the destination from a project-level record instead of asking each run
 - [ ] design skill integrated with Beads, this requires a Beads configuration in the application
 - [ ] implement skill integrated with Beads, including the concept transition to `Active`
 - [ ] test skill
 - [ ] scaffold skill for Flutter projects
