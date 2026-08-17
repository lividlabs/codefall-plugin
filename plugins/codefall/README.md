codefall
--------

**Opinionated skills for the software development lifecycle.**

This directory is the installable plugin. Everything above it — the marketplace catalog, the release
tooling, the repo README — is packaging and does not ship to users.

| Skill | Does |
| --- | --- |
| [`scaffold`](skills/scaffold/SKILL.md) | Start a new project on the Clean + package-by-component stance: ratified ADRs, scoped `AGENTS.md`, optionally project files and boundary lint. |

Skills are explicitly invoked and carry `disable-model-invocation: true`, so none fire on their own.

See the [repository README](../../README.md) for the architectural stance, the surface catalog, and
installation instructions.

## License

[MIT](LICENSE) — Copyright (c) 2026 Livid Labs, LLC, authored by Dave Jensen.

The templates under `skills/scaffold/templates/`, and everything `scaffold` copies from them into
your project, are additionally available under [0BSD](LICENSE): no attribution, no notice, no
obligation. Your architecture documents are yours.
