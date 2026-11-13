# code-index skill

A generic, text-only skill for AI coding agents: create, refresh, and validate a
code index / code navigation document for any repository, regardless of language
or framework.

## What it covers

- Discovering the repository's real architecture from manifests and source.
- Building a hierarchy: unit/package → feature → entry point → key files.
- Merging into an existing index without destroying hand-written prose.
- Validating paths, links, and entry-point coverage.
- Maintaining the index alongside code changes.

## Install

Install the whole repository:

```bash
npx skills add <owner>/<repo>
```

Install only this skill:

```bash
npx skills add <owner>/<repo> -s code-index
```

## Layout

```text
skills/
└── code-index/
    └── SKILL.md
```

## License

MIT
