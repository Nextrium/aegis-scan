# Aegis scan

A reusable GitHub Actions workflow for [Nextrium Aegis Review](https://aegis.nextrium.org). It runs
security scanners on your commit and uploads the results to Aegis. The next `/aegis audit` of that
commit triages them alongside Aegis's AI reviewers.

| Tool | Version | What it checks |
| --- | --- | --- |
| [Opengrep](https://github.com/opengrep/opengrep), with the Aegis rule pack | v1.30.2 | Injection, open redirect, SSRF, path traversal, eval, JWT "none" |
| [OSV-Scanner](https://github.com/google/osv-scanner) | v2.7.0 | Dependencies with known vulnerabilities |
| [Gitleaks](https://github.com/gitleaks/gitleaks) | v8.30.1 | Secrets committed in the checked-out files |
| [zizmor](https://github.com/zizmorcore/zizmor) | v1.31.1 | Unsafe GitHub Actions workflows |

Each tool is downloaded from its official release and checked against a pinned SHA-256 before it runs.

## Use it

The Aegis GitHub App must be installed on your account. Then add `.github/workflows/aegis.yml`:

```yaml
name: Aegis scan
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions: {}
jobs:
  aegis:
    permissions:
      contents: read
      id-token: write
    uses: Nextrium/aegis-scan/.github/workflows/aegis-scan.yml@<full commit SHA>
```

Pin a full commit SHA from this repository, not a branch or tag, so the scanner code you run cannot
change without you updating the pin.

## How the upload is trusted

- No secret is stored in your repository. The job requests a GitHub OIDC token for the audience `aegis.nextrium.org`.
- Aegis checks the token's signature against GitHub's published keys, its issuer, audience and expiry,
  and that the job ran this workflow.
- The repository and commit are taken from the token, so a job can only report on what it actually scanned.
- Code snippets are removed on the runner before upload. Aegis also redacts anything that looks like a credential.

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `aegis-url` | `https://aegis.nextrium.org` | Aegis base URL |
| `oidc-audience` | `aegis.nextrium.org` | Token audience; must match the Aegis deployment |
| `extra-rules` | none | Path in your repository to extra Opengrep rules |
