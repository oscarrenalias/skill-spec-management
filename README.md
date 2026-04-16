# skill-spec-management

An opinionated spec management skill for spec-driven development workflows. It provides a CLI tool (`spec.py`) and a Claude skill that enforce a structured lifecycle for feature specs — from initial draft through planning to completion.

Specs are Markdown files with YAML frontmatter. They live in a three-stage folder hierarchy (`drafts/` → `planned/` → `done/`) and are the single source of truth for what is being built, why, and when it's done. It also possible to use custom status values, e.g., "blocked", "postponed", the CLI will deal with that just fine but the skill enforces the default workflow.

## Installation

### Via APM

```bash
apm install oscarrenalias/skill-spec-management
```

APM will copy the skill into every detected target directory (`.github/`, `.claude/`, `.cursor/`, etc.).

### Via zip package (no APM required)

Each release ships a zip archive containing the skill. Download it from the [Releases page](https://github.com/oscarrenalias/skill-spec-management/releases) and unzip it into your project's skills folder:

```bash
# Claude Code
unzip skill-spec-management-vX.Y.Z.zip -d .claude/

# GitHub Copilot / Codex
unzip skill-spec-management-vX.Y.Z.zip -d .github/
```

The zip contains `skills/skill-spec-management/` and `agents/` directories, which unzip directly into the right locations. The skill and agent are picked up automatically on the next session.

## Onboarding a New Project

All spec operations are handled by the agent — just describe what you want in natural language.

### 1. Initialise the specs folder

> "Please initialise the specs folder for this project."

The agent creates `specs/drafts/`, `specs/planned/`, and `specs/done/`. If `specs/` already exists, this step can be skipped — the `create` command auto-creates any missing subdirectories.

### 2. Create your first spec

> "Create a spec for adding OAuth login support."

The agent creates a structured draft in `specs/drafts/` with a generated ID and prompts you to fill in the key sections: **Objective**, **Problems to Fix**, **Changes**, **Files to Modify**, **Acceptance Criteria**, and **Pending Decisions**.

### 3. Manage the lifecycle

> "Show me all the specs currently in draft."

> "Mark spec-a3f19c2b as planned."

> "What specs are blocked or high priority?"

The agent moves specs between `drafts/` → `planned/` → `done/` as work progresses, keeping frontmatter and filesystem location in sync.

The three built-in statuses (`draft`, `planned`, `done`) map to specific folders and drive the default workflow. The skill also accepts any custom status string — useful for tracking states like `blocked` or `postponed` that don't map to a lifecycle folder:

> "Mark spec-a3f19c2b as blocked."

> "Set the status of the auth spec to postponed."

Custom statuses update the frontmatter, and the skill will move the file to a folder corresponding to its status. If the folder does not exist, it will be automatically created.

### 4. Migrating existing specs

> "I have some existing spec files without frontmatter — can you migrate them?"

The agent adds a generated ID, infers the name from the first heading or filename, and sets the status from each file's current folder.


## CLI Reference

Run `spec.py` from the project root (where `specs/` lives):

```bash
python3 spec.py <subcommand> [args]
```

| Subcommand | Description |
|---|---|
| `init` | Create the `specs/drafts/`, `specs/planned/`, `specs/done/` folder structure |
| `create <title>` | Create a new spec in `specs/drafts/` with a generated ID and standard template; auto-creates lifecycle subdirs if missing |
| `list` | List all specs with ID, status, priority, complexity, and name |
| `list --status <s>` | Filter by status (`draft`, `planned`, `done`) |
| `list --tag <tag>` | Filter by tag |
| `list --priority <p>` | Filter by priority (`high`, `medium`, `low`) |
| `show <spec>` | Print frontmatter and first 20 lines of body |
| `show --full <spec>` | Print frontmatter and complete body |
| `set status <value> <spec>` | Transition status and move file to the matching folder |
| `set priority <value> <spec>` | Set `priority` field (`high`, `medium`, `low`) |
| `set tags <tag1,tag2> <spec>` | Replace the `tags` list |
| `set description <text> <spec>` | Set the `description` field |
| `set feature-root <id> <spec>` | Link spec to a feature root bead ID |
| `migrate <spec>` | Add frontmatter to a legacy spec that has none |
| `remove <spec>` | Delete a spec file (prompts confirmation for non-draft specs) |
| `remove --force <spec>` | Delete without confirmation |

`<spec>` can be a full spec ID (e.g. `spec-a3f19c2b`) or a partial filename substring.

`spec.py` has no external dependencies — it runs with the Python standard library only.