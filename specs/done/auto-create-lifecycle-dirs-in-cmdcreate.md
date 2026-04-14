---
name: Auto-create lifecycle dirs in cmd_create
id: spec-791d01a3
description: Fix cmd_create to auto-create missing lifecycle subdirectories instead of exiting with an error
dependencies: []
priority: high
complexity: low
status: done
tags:
- bugfix
scope:
  in: null
  out: null
feature_root_id: B-723fabbf
---
# Auto-create lifecycle dirs in cmd_create

## Objective

`spec.py create` currently calls `_require_lifecycle_dirs()`, which exits with an error if any of `specs/drafts/`, `specs/planned/`, or `specs/done/` are missing. This forces users to run `spec init` first (or manually create the dirs), even though `create` only needs `specs/drafts/` to write a file.

The fix is to replace the guard with `os.makedirs(d, exist_ok=True)` for each lifecycle dir inside `cmd_create`, so the directories are silently created on demand.

## Key Decisions

- Auto-create all three lifecycle dirs (not just `drafts/`) to keep the workspace consistent.
- `_require_lifecycle_dirs()` can remain for other callers; `cmd_create` simply stops using it.

## Acceptance Criteria

- `spec.py create "My spec"` succeeds when `specs/` exists but subdirs are absent.
- `spec.py create "My spec"` succeeds when `specs/` does not exist at all? No — `_require_specs_dir()` guard remains; `specs/` must still exist.
- Existing behaviour is unchanged when all dirs already exist.
- New test case: create spec with only `specs/` present (no subdirs) → succeeds.

## Owners
