# Lights-Out Software Factory (LOSF)

A fully automated software development pipeline powered by GitHub Actions and [Goose](https://github.com/block/goose) AI recipes. When a human approves an issue, the LOSF pipeline automatically researches, architects, implements, reviews, and creates a pull request.

## How It Works

```
Issue filed -> Human reviews -> Adds "approved" label
                                      |
                                      v
                            GitHub Action triggers
                                      |
                    +-----------------+
                    v
            [Architect Phase]
            - Researches codebase
            - Analyzes requirements
            - Writes implementation plan
                    |
                    v
            [Dev-QE Loop]
            - Developer implements changes
            - QE reviews + runs tests
            - Iterates up to 10 rounds
                    |
                    v
            [Pull Request Created]
            - Links to original issue
            - Includes QE verdict
            - Ready for human review
```

## Pipeline Components

### 1. Development Pipeline (`losf-dev-pipeline.yml`)

Triggered when an issue receives the `approved` label. Runs the full architect -> dev -> QE cycle and creates a PR.

### 2. Merge Conflict Resolution (`losf-merge-conflict.yml`)

Runs on a schedule (every 4 hours), on push to main, or manually. Detects and automatically resolves merge conflicts in open PRs.

### 3. Goose Recipes

| Recipe | Purpose |
|--------|---------|
| `dev-pipeline.yaml` | Main orchestrator — runs architect then dev-qe-loop |
| `architect.yaml` | Analyzes requirements, researches codebase, writes implementation plan |
| `dev-qe-loop.yaml` | Developer implements, QE reviews, iterates until approved |
| `merge-conflict.yaml` | Resolves merge conflicts intelligently |

## Filing Issues

Use the provided issue templates:

- **Feature Request** — Describe the feature, list acceptance criteria
- **Bug Report** — Describe the bug, steps to reproduce, expected behavior

Well-structured issues produce better automated implementations.

## Setup

### Prerequisites

1. A GitHub repository with Actions enabled
2. A Gemini API key

### Configuration

1. **Add repository secret:** Go to Settings > Secrets > Actions and add `GOOGLE_API_KEY` with your Gemini API key.

2. **Create the `approved` label:** Go to Issues > Labels and create a label named `approved`.

3. **Enable PR creation by Actions:** Go to Settings > Actions > General and enable "Allow GitHub Actions to create and approve pull requests".

4. **Branch protection (recommended):** Require PR reviews on `main` so AI-generated PRs still receive human review before merging.

## Architecture

### State Passing

Subrecipes communicate via the filesystem (`.goose/scratchpad/`):

| File | Writer | Reader | Purpose |
|------|--------|--------|---------|
| `issue-body.md` | GitHub Action | Architect | Issue description |
| `architect-plan.md` | Architect | Dev-QE Loop | Implementation plan |
| `dev-changes.md` | Developer | QE | Change summary per round |
| `qe-feedback.md` | QE | Developer | Review feedback per round |
| `qe-verdict.md` | QE (final) | GitHub Action | Final approval for PR body |

Scratchpad files are gitignored and do not appear in commits or PRs.

### Goose Provider Configuration

The pipeline configures Goose to use Gemini:

```yaml
provider: gemini
model: gemini-3.1-pro-preview
keyring: false
```

The `GOOGLE_API_KEY` environment variable is set from the GitHub secret.

## License

See [LICENSE](LICENSE) for details.
