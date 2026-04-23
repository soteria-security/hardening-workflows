# hardening-workflows

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

A collection of GitHub Actions workflows for automating security hardening across your organization — including supply chain protection, action pinning, and CI/CD policy enforcement.

---

## Workflows

| Workflow | Description |
|---|---|
| [pin-actions-to-sha](workflows/pin-actions-to-sha/) | Pins all GitHub Actions `uses:` references org-wide to their full-length commit SHAs and opens a pull request per repo with the changes |

---

## Design Philosophy

Every workflow in this repository is built around the following principles:

- **No personal access tokens.** All cross-repo operations use a GitHub App installation token, eliminating the risk of PAT leakage or single-user dependency.
- **Signed commits by default.** Commits are created via the GitHub Contents API rather than `git push`, which means GitHub automatically signs them — satisfying branch protection rules that require verified commits (vigilant mode).
- **Dry-run first.** Workflows default to dry-run mode, letting you preview every proposed change before anything is committed or a PR is opened.
- **Safe to re-run.** All operations are idempotent — running a workflow multiple times on the same repo produces the same result.

---

## Prerequisites

All workflows in this repository authenticate using a **GitHub App** rather than a personal access token. Each workflow's own README documents the exact permissions required and provides step-by-step setup instructions.

At a high level you will need to:

1. Create a GitHub App in your org with the permissions listed in the workflow's README.
2. Install the app on your org (or on the specific repos you want to target).
3. Add the App ID and private key as org-level Actions secrets in the repo where the workflow lives.

See the individual workflow README for full details.

---

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

---

## Acknowledgements

This repository includes a workflow that uses [**ratchet**](https://github.com/sethvargo/ratchet) by [@sethvargo](https://github.com/sethvargo) to resolve and pin action references to their commit SHAs. Ratchet is downloaded at runtime from its [GitHub Releases](https://github.com/sethvargo/ratchet/releases) and is not bundled in this repository. It is licensed under the [Apache License 2.0](https://github.com/sethvargo/ratchet/blob/main/LICENSE).

---

## License

This project is licensed under the [Apache License 2.0](LICENSE).
