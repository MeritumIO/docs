+++
title = "Meritum"
+++

Meritum is a collection of composable PHP libraries built on the principle of _natural aristoi_ — components that earn their place through demonstrated value, not convention. No all-or-nothing frameworks, no hidden opinions: each package does one thing, has hard boundaries, and can be replaced when something better comes along.

[Read the full philosophy →](https://dev.to/georgeff/natural-aristoi-h6j)

[View the GitHub organization →](https://github.com/MeritumIO)

## Get Started

The fastest path to a working application is a scaffold. Both scaffolds ship with a pre-wired kernel, a `devenv.nix`, and a placeholder test suite.

**Sage — CLI Application**

```bash
composer create-project meritum/sage my-app
```

Pre-wired `meritum/cli` kernel, devenv shell, ready to add commands.

**Virtus — HTTP API**

```bash
composer create-project meritum/virtus my-api
```

Pre-wired `meritum/http` kernel, built-in server starts on port 8000.

## Explore the Docs

- **[georgeff/kernel](/docs/kernels/kernel/)** — dependency injection, modules, and the boot lifecycle. Start here to understand how everything fits together.
- **[meritum/http](/docs/kernels/http/)** — HTTP kernel, routing, and PSR-15 middleware.
- **[meritum/cli](/docs/kernels/cli/)** — console kernel and command registration.
- **[meritum/validation](/docs/packages/validation/)** — attribute-driven validation with a fluent rule DSL.
- **[meritum/database](/docs/packages/database/)** — active-record model, repository pattern, and casting.
- **[meritum/migrations](/docs/packages/migrations/)** — schema migration runner with a CLI interface.
- **[meritum/serialization](/docs/packages/serialization/)** — pluggable object serialization with item, collection, and pagination support.
- **[meritum/logger](/docs/packages/logger/)** — minimal PSR-3 logger that writes newline-delimited JSON to stdout.
- **[meritum/structured-logging](/docs/packages/structured-logging/)** — domain exception model, translation pipeline, PSR-3 reporting, and correlation ID enrichment.
- **[meritum/http-exception-handler](/docs/packages/http-exception-handler/)** — translates exceptions into structured JSON error responses.
