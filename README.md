# Skill Pack GitHub Action

Package Agent Skills into `.skill` bundles, upload to GitHub releases, and announce to the mpak registry.

## Features

- **Auto-discovery**: Finds all skills in your repository by looking for `SKILL.md` files
- **Validation**: Validates frontmatter against the Agent Skills specification
- **Bundling**: Creates `.skill` zip archives ready for distribution
- **Release upload**: Uploads bundles to GitHub releases
- **Registry announce**: Announces skills to mpak.dev via OIDC (no API keys needed)

## Usage

### Basic Usage (Release Trigger)

```yaml
name: Publish Skills

on:
  release:
    types: [published]

permissions:
  contents: write  # For uploading to releases
  id-token: write  # For OIDC authentication

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Pack and publish skills
        uses: NimbleBrainInc/skill-pack@v1
```

### Multi-Skill Repository

If your repository contains multiple skills, the action will discover and process all of them:

```
my-skills-repo/
├── code-reviewer/
│   └── SKILL.md
├── test-writer/
│   └── SKILL.md
└── doc-generator/
    └── SKILL.md
```

### Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `directory` | Directory to search for skills | `.` |
| `pattern` | Glob pattern to find skill directories | `**/SKILL.md` |
| `build` | Whether to build .skill bundles | `true` |
| `upload` | Whether to upload to GitHub release | `true` |
| `announce` | Whether to announce to registry | `true` |
| `announce-url` | Registry announce endpoint | `https://api.mpak.dev/v1/skills/announce` |
| `fail-on-warning` | Fail if validation warnings found | `false` |

### Outputs

| Output | Description |
|--------|-------------|
| `skills-found` | Number of skills discovered |
| `skills-packed` | Number of skills successfully packed |
| `skills-announced` | Number of skills announced to registry |
| `bundle-paths` | JSON array of bundle paths |

## Skill Requirements

Each skill must have a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: my-skill
description: What this skill does and when to use it

# Optional fields
license: MIT
compatibility: Claude Code only
allowed-tools: Read Grep Bash

# Discovery metadata (recommended)
metadata:
  version: 1.0.0
  category: development
  tags:
    - code-review
    - testing
  triggers:
    - "review this code"
    - "check for bugs"
  surfaces:
    - claude-code
  author:
    name: Your Name
    url: https://github.com/yourname
---

# My Skill

Instructions for the skill...
```

### Required Fields

- `name`: Must match the directory name (lowercase, hyphens allowed)
- `description`: What the skill does and when to use it

### Recommended Fields

- `metadata.version`: Required for registry publishing
- `metadata.category`: For filtering (development, writing, research, etc.)
- `metadata.tags`: For discovery
- `metadata.triggers`: Phrases that activate this skill

## OIDC Authentication

The action uses GitHub's OIDC tokens to authenticate with the registry. This means:

1. No API keys to manage
2. Package scope must match your GitHub org/user
3. Only your workflows can publish your skills

The package name will be scoped to your GitHub organization or username:
- Org `NimbleBrainInc` → `@nimblebraininc/skill-name`
- User `johndoe` → `@johndoe/skill-name`

## Examples

### Validate Only (No Publish)

```yaml
- uses: NimbleBrainInc/skill-pack@v1
  with:
    build: false
    upload: false
    announce: false
```

### Custom Directory

```yaml
- uses: NimbleBrainInc/skill-pack@v1
  with:
    directory: ./skills
```

### Strict Validation

```yaml
- uses: NimbleBrainInc/skill-pack@v1
  with:
    fail-on-warning: true
```

## Troubleshooting

### "Failed to get OIDC token"

Ensure your workflow has the correct permissions:

```yaml
permissions:
  id-token: write
```

### "Scope mismatch"

The skill name scope must match your GitHub organization or username. If your repo is `NimbleBrainInc/my-skills`, the skill will be published as `@nimblebraininc/skill-name`.

### "Artifact not found in release"

Make sure you're running on a `release` event and the release has been created before the action runs. The action will retry a few times to handle GitHub API propagation delays.

## Related

- [Agent Skills Specification](https://agentskills.io/specification)
- [mpak Registry](https://mpak.dev)
- [mpak CLI](https://github.com/NimbleBrainInc/mpak-cli)
