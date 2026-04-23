# pin-actions-to-sha

Pins all GitHub Actions `uses:` references across every non-archived repository in a GitHub org from mutable semver tags to immutable full-length commit SHAs, then opens a pull request per repository with the changes.

**Example of what it does:**

```yaml
# Before
uses: actions/checkout@v4

# After
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

Pinning to a commit SHA means a compromised or hijacked tag can no longer silently alter what code runs in your pipelines. This is one of the most impactful single steps you can take to harden your CI/CD supply chain.

---

## How It Works

The workflow runs three jobs:

1. **`list-repos`** — Calls the GitHub API to enumerate all non-archived repositories in your org (or a single repo if you specify one) and outputs a JSON array used to drive a matrix.

2. **`pin-actions`** _(matrix, one job per repo)_ — For each repo:
   - Checks out the repository using the GitHub App token
   - Skips repos with no workflow files
   - Downloads and checksum-verifies the [ratchet](https://github.com/sethvargo/ratchet) binary
   - Runs `ratchet pin` against every workflow file in `.github/workflows/`
   - Rewrites ratchet's inline comments from `# ratchet:owner/action@v4` to `# v4` so Dependabot can continue proposing version updates after pinning
   - In **dry-run mode**: prints the full diff and stops — no branch, no commit, no PR
   - In **live mode**: creates a branch, commits each modified file individually via the GitHub Contents API (producing signed commits), creates any missing PR labels, and opens a pull request

3. **`summarize`** — Downloads per-repo metrics artifacts from all matrix jobs and writes a grand-total summary table to the workflow run's summary page showing unpinned counts before and after.

---

## Required GitHub App Permissions

The workflow uses a GitHub App installation token for all cross-repository operations. The app must be granted the following **repository permissions**:

| Permission | Access | Reason |
|---|---|---|
| **Metadata** | Read | Required by GitHub for all apps; used to read repo information and list repos |
| **Contents** | Read & Write | To read workflow files and commit pinned versions via the Contents API |
| **Workflows** | Read & Write | Required by GitHub to push changes to files inside `.github/workflows/` |
| **Pull requests** | Read & Write | To open pull requests with the pinned changes |
| **Issues** | Read & Write | To create and update the `dependencies` and `github_actions` PR labels |

> **Note:** The **Workflows** permission is a separate, explicit permission in GitHub Apps. Without it, commits to `.github/workflows/` paths will be rejected even if Contents is set to Read & Write.

---

## Setting Up the GitHub App

### Step 1 — Create the app

1. Go to your org's **Settings** → **Developer settings** → **GitHub Apps** → **New GitHub App**
2. Fill in the required fields:
   - **GitHub App name:** Something like `pin-actions-bot` (must be globally unique on GitHub)
   - **Homepage URL:** Your org URL, e.g. `https://github.com/your-org`
3. Under **Webhook**, uncheck **Active** — this app does not need webhooks
4. Under **Repository permissions**, set the following (leave everything else as **No access**):

   | Permission | Setting |
   |---|---|
   | Contents | **Read & Write** |
   | Issues | **Read & Write** |
   | Metadata | **Read** _(auto-selected, cannot be changed)_ |
   | Pull requests | **Read & Write** |
   | Workflows | **Read & Write** |

   Then scroll down to **Organization permissions** and set the following:

   | Permission | Setting |
   |---|---|
   | Members | **Read** |

   > **Why Members: Read?** The workflow calls `GET /orgs/{org}/repos` to enumerate all repositories in the org. GitHub requires the **Members: Read** organization-level permission for a GitHub App installation token to list repositories across an org, including private ones. Without it, the `list-repos` job will return an empty or incomplete repository list.

5. Under **Where can this GitHub App be installed?**, select **Only on this account**
6. Click **Create GitHub App**

### Step 2 — Note the App ID

After the app is created, you will land on its settings page. Copy the **App ID** shown near the top — you will need it in Step 5.

### Step 3 — Generate a private key

1. Scroll to the bottom of the app settings page
2. Click **Generate a private key**
3. A `.pem` file will be downloaded automatically — store it securely

### Step 4 — Install the app on your org

1. In the app settings, click **Install App** in the left sidebar
2. Click **Install** next to your org
3. Choose **All repositories** to allow the workflow to target your entire org, or select specific repositories if you want to limit scope
4. Click **Install**

### Step 5 — Add the secrets to your repo

The workflow reads two secrets. These must be added to the **repository** where this workflow file lives (or as org-level secrets if you prefer to share them across repos).

Navigate to the repo **Settings** → **Secrets and variables** → **Actions** → **New repository secret** and create both of the following:

| Secret name | Value |
|---|---|
| `PIN_ACTIONS_APP_ID` | The App ID you copied in Step 2 |
| `PIN_ACTIONS_APP_PRIVATE_KEY` | The full contents of the `.pem` file downloaded in Step 3 |

> **Important:** The secret names must match exactly as shown above — they are referenced by name in the workflow source.

---

## Installation

1. Copy `pin-actions-to-sha.yml` into the `.github/workflows/` directory of the repository where you want to run it
2. Complete the GitHub App setup above and add both secrets
3. Go to **Actions** → **Pin Workflow Actions to Commit SHA (Org-Wide)** → **Run workflow**

---

## Usage

Trigger the workflow manually via **Actions** → **Run workflow**. You will be prompted for three inputs:

| Input | Required | Default | Description |
|---|---|---|---|
| `org` | **Yes** | — | The GitHub org to target (e.g. `my-org`) |
| `dry_run` | No | `true` | When `true`, logs all proposed changes but creates no branches, commits, or PRs. **Always run in dry-run mode first.** |
| `target_repo` | No | _(blank)_ | Name of a single repo to process (e.g. `my-repo`). Leave blank to process all non-archived repos in the org. |

### Recommended first-run sequence

1. Run with `dry_run: true` and `target_repo: <one repo name>` — review the diff in the job log
2. Run with `dry_run: true` and `target_repo` blank — review the summary table across all repos
3. Run with `dry_run: false` — PRs will be opened in every repo with unpinned actions

---

## Keeping ratchet Up to Date

The ratchet version is pinned in the workflow via the `RATCHET_VERSION` env variable:

```yaml
env:
  RATCHET_VERSION: "0.11.4"
```

Check [github.com/sethvargo/ratchet/releases](https://github.com/sethvargo/ratchet/releases) periodically for new releases and update this value accordingly. The workflow verifies the binary against the published SHA-512 checksum before executing it.

---

## Important Notes

- **256-repo matrix limit:** GitHub Actions matrices are capped at 256 entries per dimension. If your org has more than 256 non-archived repositories, the `list-repos` job will need to be extended to support chunking.
- **Signed commits:** Commits are created via the GitHub Contents API using the App token. GitHub automatically signs these commits, satisfying branch protection rules that require verified commits (vigilant mode). A direct `git push` would not produce signed commits.
- **Reusable workflows are also pinned:** `ratchet` pins reusable workflow `uses:` calls (e.g. `uses: org/repo/.github/workflows/file.yml@ref`) in addition to action references. If you want to keep specific references at a branch or tag ref, add a `# ratchet:exclude` comment to that line before running.
- **Dependabot compatibility:** The workflow rewrites ratchet's inline comments from `# ratchet:owner/action@v4` format to `# v4` format, which is what Dependabot requires to recognise and propose version updates on pinned lines.
- **Idempotent:** If all actions in a repo are already pinned, no branch or PR is created. If the workflow is re-run and a PR for the branch already exists, the existing PR URL is surfaced rather than failing the step.

---

## Run Summary

After each run, a summary table is written to the workflow run's **Summary** page showing, for each repo:

- **Unpinned before** — the number of unpinned `uses:` references found before ratchet ran
- **Newly pinned** — the number of references that were pinned in this run

A grand total row is included at the bottom.

---

## Acknowledgements

This workflow uses [**ratchet**](https://github.com/sethvargo/ratchet) by [@sethvargo](https://github.com/sethvargo), licensed under the [Apache License 2.0](https://github.com/sethvargo/ratchet/blob/main/LICENSE). Ratchet is downloaded at runtime and is not bundled in this repository.

---

## License

[Apache License 2.0](../../LICENSE)
