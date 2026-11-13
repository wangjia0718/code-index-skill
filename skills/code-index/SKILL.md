---
name: code-index
description: Use when asked to create, refresh, or validate a code index / code navigation document for any repository, or after adding, renaming, moving, or deleting modules, entry points, models, services, or migrations and the index must stay accurate.
---

# Code Index

## Overview

A code index is a **navigation map**, not a file dump. It answers "where does
feature X live?" in a few hops: deployable unit / module → feature → entry
point → key files and symbols.

**Core principle:** derive the structure from the repository in front of you.
Never assume a language, framework, directory name, or document format.

## When to Use

- Creating a code index / code navigation document from scratch.
- Refreshing an existing index after modules, entry points, models, services,
  or migrations change.
- A full-repo scan is about to happen while looking for a feature entry point —
  check the index first.

## Workflow

### 1. Learn the repository before writing

Read, in this order:

1. `README`, `AGENTS.md`, `CLAUDE.md`, contribution docs.
2. Workspace and build manifests: `package.json`, `pnpm-workspace.yaml`,
   `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`,
   `composer.json`, `Gemfile`, `*.sln`, `*.csproj`, `tsconfig.json`.
3. Existing architecture/index docs (`CODE_INDEX.md`, `docs/code-index.md`,
   `ARCHITECTURE.md`, `CODEBASE.md`).

From these, answer:

- Is this a single app or a monorepo? What are the deployable units?
- What language(s) and framework(s) are in use?
- Where do source, tests, scripts, migrations, and generated code live?
- What naming conventions does the project already use?

### 2. Inventory the tree

Start broad, then narrow. Useful commands:

```bash
git ls-files | sed 's#/.*##' | sort | uniq -c | sort -rn   # top-level shape
find . -maxdepth 2 -name package.json -o -name pyproject.toml -o -name go.mod
rg --files -g '!node_modules' -g '!dist' -g '!build' -g '!vendor'
```

Ignore generated, vendored, cached, and build directories: `.git`,
`node_modules`, `.venv`, `venv`, `dist`, `build`, `target`, `out`, `coverage`,
`__pycache__`, `.next`, `.nuxt`, `.turbo`, `.cache`, `vendor`, `generated`.

### 3. Find entry points by convention

Search for the project's own architectural seams instead of imposing a template.

| Concern | Typical signals |
| --- | --- |
| HTTP/API surface | `route`, `router`, `urls`, `controller`, `handler`, `endpoint`, `page` (frontend), `route.ts` |
| Business logic | `service`, `usecase`, `use-case`, `domain`, `manager`, `provider` |
| Persistence | `model`, `entity`, `schema`, `repository`, `dao`, `migration`, `versions` |
| Async/work | `task`, `job`, `worker`, `consumer`, `cron`, `scheduler` |
| Integrations | `client`, `adapter`, `gateway`, `integration`, `connector` |
| Shared code | `lib`, `common`, `shared`, `core`, `utils`, `packages/` |

Verify candidates by opening the files. A file called `service.ts` may be a
utility; a file called `orders.ts` may be the real route module.

### 4. Build the index hierarchy

Use the repository's own boundaries, then feature boundaries inside them:

```
deployable unit / package / module
  └─ feature / subdomain
       ├─ entry point (routes/controllers/handlers)
       ├─ business logic (services/use cases)
       ├─ persistence (models/entities/migrations)
       └─ tests (only if useful for navigation)
```

Each row should point to an **entry path**, not an exhaustive file list:

```markdown
### Orders
| Entry | Path | Notes |
| --- | --- | --- |
| HTTP routes | `src/routes/orders.*` | order CRUD, approval |
| Domain logic | `src/services/order/` | pricing, validation |
| Persistence | `src/models/order.*` | `orders`, `order_items` |
| Migration | `src/migrations/` | latest relevant migration |
```

For monorepos, start with a repository map and then one section per unit:

```markdown
## Repository Map
| Unit | Path | Responsibility |
| --- | --- | --- |
| API | `apps/api/` | HTTP backend |
| Web | `apps/web/` | Frontend |
| Shared | `packages/shared/` | cross-app types and utilities |
```

Adapt headings and language to any existing document; do not force a new format
onto a project that already has one.

### 5. Merge, do not overwrite

If an index already exists:

- Preserve its headings, language, table shape, and hand-written explanations.
- Update changed rows; add new modules; delete paths that no longer exist.
- Keep business-oriented quick-locate tables if the project uses them.
- Keep path bases explicit: either all paths are repository-relative, or state
  the base directory at the top.
- Summarize large folders (`services/`, `handlers/`, `components/`) instead of
  listing every file.

### 6. Validate

Every path in the index must exist. A language-agnostic check:

```bash
grep -oE '`[^`]+`' docs/code-index.md | tr -d '`' | sort -u |
while read -r p; do
  case "$p" in
    http*|"#"*|*"<"*|*">"*|*"*"*|*" "* ) continue ;;
  esac
  [ -e "$p" ] || echo "MISSING $p"
done
```

Also:

- Confirm the index mentions every primary entry-point file
  (`rg --files -g '*router*' -g '*controller*' -g '*handler*'` etc.).
- Open one new row and verify the symbol/path actually exists.
- Review `git diff -- <index>`: the change should be scoped to the index and
  must not delete most existing prose.
- If migrations are indexed, confirm the current head is the real head.

### 7. Keep it current

Treat the index as part of the change, not a separate cleanup task:

- New module/subdomain → add a row.
- Renamed/moved entry point → update paths and links.
- Deleted module → remove its rows and stale references.
- New migration → update the migration entry/head if the index tracks it.
- Changed responsibility → update the one-line description.

## What Belongs

- Deployable units, packages, and major modules with responsibilities.
- Feature/subdomain rows with entry points, models, services, and migrations.
- Cross-cutting layers: shared libraries, jobs, integrations, generated clients,
  infrastructure.
- A short "quick locate" table mapping business questions to entry points, when
  the existing index has one.

## What Does Not Belong

- Generated code, vendored dependencies, caches, build output.
- Every file or every function.
- Guessed paths or guessed symbols.
- Framework templates that do not match the repository's actual layout.
- Duplicated content already maintained in another authoritative index.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Choosing a format before reading the repo | Read manifests, docs, and existing index first |
| Assuming FastAPI/Django/Next.js/Rails structure | Detect the stack and follow its real conventions |
| Overwriting an existing index | Merge into its structure; preserve labels and prose |
| Listing every file | Index entry points; summarize directories |
| Mixing path bases | State the base once and keep every path consistent |
| Linking to guessed symbols | Open the file and confirm |
| Leaving stale paths after moves/deletes | Validate every path and remove dead entries |
| Indexing generated/vendor code | Exclude build, cache, and dependency directories |
| Skipping the index in a code change | Update it in the same change |

## Quality Gate

Before claiming the index is complete, all of these must be true:

- The structure reflects the repository's actual units and conventions.
- Every referenced path exists.
- Every primary entry point in the changed area is represented.
- No existing explanation or quick-locate entry was removed without reason.
- The diff is scoped to the index.
- A developer can go from a business feature name to its entry point in one or
  two hops.
