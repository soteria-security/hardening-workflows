# Contributing to hardening-workflows

Thank you for your interest in contributing! This repository exists to help the broader security and DevSecOps community harden their GitHub Actions supply chains. Contributions from practitioners who are solving real problems in this space are especially welcome.

Please take a few minutes to read this guide before opening a pull request.

---

## Ways to Contribute

- **Add a new workflow** — share a hardening workflow your team has found valuable
- **Improve an existing workflow** — bug fixes, edge case handling, better error messages, performance improvements
- **Improve documentation** — clearer setup steps, better examples, typo fixes
- **Report a bug** — open an issue describing the problem and steps to reproduce it
- **Suggest an idea** — open an issue to discuss a new workflow before building it

---

## Proposing a New Workflow

Before writing a new workflow, **please open an issue first** to describe:

1. What the workflow does
2. What security problem it solves or what hardening goal it achieves
3. What GitHub APIs, permissions, or third-party tools it depends on
4. Whether it operates at repo-level, org-level, or both

This gives maintainers a chance to provide early feedback, avoid duplication of effort, and ensure the workflow fits the scope of this repository before you invest time building it.

---

## Submitting a Pull Request

1. **Fork** the repository and create a branch from `main`
2. Make your changes following the standards below
3. Open a pull request against `main` with a clear title and description explaining what the change does and why

All pull requests require at least one maintainer review before merging.

---

## Workflow Standards

All workflows contributed to this repository must follow these standards:

### Action pinning — dogfood the purpose of this repo

Every `uses:` reference in a contributed workflow file **must** be pinned to a full-length commit SHA with the semver tag preserved as an inline comment. Mutable tag references will be requested to be changed during review.

```yaml
# ✅ Correct
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4

# ❌ Not accepted
uses: actions/checkout@v4
```

### SPDX license headers

Every workflow YAML file must include the following SPDX header comments as the first two lines of the file:

```yaml
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: <year> <your name or org> (https://github.com/<your-handle>)
```

### File structure

Each workflow should live in its own subdirectory under `workflows/` and include a `README.md` with full setup and usage instructions:

```
workflows/
└── your-workflow-name/
    ├── your-workflow-name.yml
    └── README.md
```

The workflow `README.md` should follow the same structure as the existing [pin-actions-to-sha README](workflows/pin-actions-to-sha/README.md) and include at minimum:

- An overview of what the workflow does and why it matters for security
- A description of how it works (jobs, key steps)
- A complete GitHub App or authentication setup section with required permissions
- An inputs reference table
- Any important caveats or limitations

### GitHub App over PATs

Workflows should use a GitHub App installation token for cross-repository or cross-org operations rather than a personal access token. PAT-based approaches create single-user dependencies and leak privileges. If your workflow requires elevated access, document the minimal required permissions clearly.

### Dry-run mode

Where practical, workflows should support a `dry_run` input that defaults to `true`, allowing users to preview all proposed changes before anything is committed or mutated.

### Permissions blocks

Every job must declare an explicit `permissions:` block. Use the principle of least privilege — only request permissions that the job actually uses.

---

## Documentation Standards

- Write for an infosec/DevSecOps audience that may not be deeply familiar with GitHub Actions internals
- Include concrete examples — show before/after where relevant
- Explain *why* a step or permission is needed, not just *what* it does
- Link to relevant GitHub documentation where appropriate

---

## Reporting a Security Vulnerability

Please **do not** open a public issue to report a security vulnerability in a workflow in this repository. Instead, use [GitHub's private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) feature for this repository.

---

## License

By contributing to this repository, you agree that your contributions will be licensed under the [Apache License 2.0](LICENSE). Please ensure you have the right to contribute any code or content you submit.
