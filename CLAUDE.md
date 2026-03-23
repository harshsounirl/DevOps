# CLAUDE.md

This file provides guidance for AI assistants (Claude and others) working in this repository.

## Repository Overview

**DevOps** is a development and operations automation repository owned by `harshsounirl`. It is intended to house infrastructure-as-code, CI/CD pipelines, automation scripts, and operational tooling.

- **Remote**: `http://local_proxy@127.0.0.1:41715/git/harshsounirl/DevOps`
- **Default branch**: `master`
- **Primary language**: To be determined as the project grows (likely shell, Python, YAML, HCL, or a combination)

---

## Repository Structure (Evolving)

As the project grows, the expected layout is:

```
DevOps/
├── CLAUDE.md          # This file — AI assistant guidance
├── README.md          # Human-facing project overview
├── .github/           # GitHub Actions workflows and PR/issue templates
│   └── workflows/
├── scripts/           # Automation and utility shell/Python scripts
├── infra/             # Infrastructure-as-code (Terraform, Pulumi, CDK, etc.)
├── k8s/               # Kubernetes manifests
├── docker/            # Dockerfiles and Compose files
├── ci/                # CI/CD pipeline definitions (if not in .github/)
└── docs/              # Operational runbooks and architecture diagrams
```

When adding new files, place them in the most semantically appropriate directory above. Create the directory if it does not yet exist.

---

## Development Workflow

### Branching

- **`master`** is the stable, production-ready branch. Do not push directly to `master` without a reviewed PR.
- Feature and fix branches follow this naming convention:
  - `feature/<short-description>` — new functionality
  - `fix/<short-description>` — bug fixes
  - `chore/<short-description>` — maintenance, dependency updates, refactoring
  - `claude/<task-id>` — branches created by AI assistants (e.g. `claude/add-claude-documentation-BN6WE`)

### Commits

- Write clear, imperative commit messages: `Add Terraform module for VPC`, not `added stuff`.
- Keep commits atomic — one logical change per commit.
- Reference issue numbers when applicable: `Fix health-check timeout (#42)`.

### Pull Requests

- Open a PR for every change targeting `master`.
- Include a summary of *what* changed and *why*.
- Ensure CI passes before requesting review.

---

## Key Conventions

### Shell Scripts

- Use `#!/usr/bin/env bash` as the shebang line.
- Enable strict mode at the top of every script:
  ```bash
  set -euo pipefail
  ```
- Quote all variable expansions: `"${VAR}"`.
- Use `snake_case` for variable and function names.
- Provide a brief usage comment at the top of each script.

### Python Scripts

- Target **Python 3.10+**.
- Use `snake_case` for functions/variables, `PascalCase` for classes.
- Format with `black` and lint with `ruff` or `flake8`.
- Keep dependencies in a `requirements.txt` or `pyproject.toml`.

### Infrastructure-as-Code

- **Terraform**: Use workspaces or separate state backends per environment (`dev`, `staging`, `prod`). Store state remotely (S3 + DynamoDB lock, or equivalent). Run `terraform fmt` before committing.
- **Docker**: Pin base image tags to a specific digest or version — never use `latest` in production images.
- **Kubernetes**: Use namespaces to isolate environments. Label all resources with `app`, `env`, and `version`.

### YAML / CI Pipelines

- Use 2-space indentation.
- Anchor repeated blocks with YAML aliases where supported.
- Store secrets in a secrets manager (GitHub Secrets, Vault, AWS SSM) — never hardcode credentials.

---

## CI/CD

When pipelines are added, the standard stages are:

1. **Lint** — static analysis and formatting checks
2. **Test** — unit and integration tests
3. **Build** — artifact or container image build
4. **Deploy (staging)** — automated deploy to a non-production environment
5. **Deploy (production)** — gated deploy requiring manual approval or a green staging run

---

## Security Guidelines

- Never commit secrets, tokens, passwords, or private keys. Use `.gitignore` and pre-commit hooks (e.g., `detect-secrets`, `git-secrets`) to enforce this.
- Rotate any credential that is accidentally committed immediately.
- Apply the principle of least privilege to all IAM roles, service accounts, and API tokens.
- Scan container images for vulnerabilities (Trivy, Grype, or Snyk) as part of CI.

---

## Testing

- Scripts and modules should include tests where feasible (e.g., `bats` for Bash, `pytest` for Python).
- Infrastructure modules should include `terraform validate` and, where possible, `terratest` checks.
- Tests live alongside the code they cover or in a top-level `tests/` directory that mirrors the source tree.

---

## Common Commands

These commands will be relevant once the project structure is populated:

```bash
# Validate all Terraform configurations
terraform validate

# Format Terraform files
terraform fmt -recursive

# Run Python tests
pytest tests/

# Lint shell scripts
shellcheck scripts/**/*.sh

# Build a Docker image
docker build -t <image-name>:<tag> -f docker/Dockerfile .
```

---

## AI Assistant Notes

- This repository is minimal at creation time. When adding new capabilities, follow the directory layout and conventions above.
- Prefer small, focused commits over large sweeping changes.
- When in doubt about a convention not covered here, match the style of the surrounding code.
- Do not push to `master` directly — always use the designated feature/claude branch and open a PR.
- Update this `CLAUDE.md` whenever significant new patterns, tools, or workflows are introduced.
