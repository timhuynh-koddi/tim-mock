# Repository Guidelines

## Project Structure & Module Organization

This repository contains configuration and JSON fixture data for a generic mock HTTP server. There is no application source tree, package manifest, or in-repo test suite.

- `mock.json` defines the mock server port, timeouts, log level, and literal endpoint paths.
- `searchads360_*.json` files are response fixtures for Search Ads 360 endpoints.
- `CLAUDE.md` documents the current mock behavior and verification approach.
- `README.md` is a minimal usage note for test customer IDs.

When adding an endpoint, add a new `endpoints[]` entry in `mock.json` and point `jsonPath` at a dedicated fixture file.

## Build, Test, and Development Commands

There is no build step. Validate JSON before handing off changes:

```bash
jq empty mock.json searchads360_*.json
```

Run the external mock server with `mock.json` using the server tool configured in your environment. Then verify routes directly:

```bash
curl http://localhost:3333/v0/customers:listAccessibleCustomers
curl -X POST http://localhost:3333/v0/customers/<customer_id>/searchAds360:search -d '{"query":"..."}'
```

Use `git diff --check` before committing to catch whitespace issues.

## Coding Style & Naming Conventions

Keep JSON fixtures formatted with two-space indentation. Preserve Google JSON API conventions: `int64` values such as `id`, `clicks`, `costMicros`, and `totalResultsCount` should be strings, not numbers.

Name new fixtures with the existing pattern, for example `searchads360_<resource>_search.json` or `searchads360_<resource>_search_<customer_id>.json`. Keep customer IDs synchronized between endpoint paths, resource names, and fixture bodies.

## Testing Guidelines

Testing is manual. Validate all changed JSON with `jq`, then curl every affected endpoint. If a fixture field is added, confirm its field mask includes the matching path where applicable. Endpoint matching is literal, so do not rely on request-body conditional behavior.

## Commit & Pull Request Guidelines

Recent commits use short, lowercase summaries such as `more mock under keyword` and `sa360 search and list customer endpoint`. Keep commits focused on one fixture or endpoint change.

Pull requests should describe the changed mock behavior, list affected endpoints and customer IDs, and include sample curl output or a concise verification note.

## Agent-Specific Instructions

Do not overwrite user edits in existing fixtures. Check `git status --short` before changing files, and treat unrelated modified files as user-owned work.
