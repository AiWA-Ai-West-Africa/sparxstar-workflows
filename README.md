# sparxstar-workflows

Reusable GitHub Actions workflows for the **SPARXSTAR platform** built by [Starisian Technologies](https://github.com/Starisian-Technologies) / [AiWA – Ai West Africa](https://github.com/AiWA-Ai-West-Africa).

---

## Claude PR Review

Every new or updated Pull Request is sent to Claude AI for review. The workflow:

1. Fetches the full PR diff (truncated to 80 KB when necessary).
2. Collects repository-specific context from `AGENTS.md`, `.github/copilot-instructions.md`, and any Markdown files found in `docs/` or `specs/`.
3. Sends everything to Claude with the SPARXSTAR platform-wide rules baked in.
4. Posts the review findings as a comment directly on the PR.

### Platform rules enforced on every PR

| Rule | Detail |
|------|--------|
| `declare(strict_types=1)` | Required in every PHP file |
| No `error_log()` | Use the platform logging pattern |
| No `SELECT *` | Column list must be explicit |
| Proprietary license only | No MIT license in any repo |
| No `wordpress/mcp-adapter` | Package does not exist |
| No type re-declaration | Types owned by `sparxstar-ouroboros-integrity` must not be redefined locally |
| No `packages/` stub dirs | Use the published package |
| No test/stub/mock files | Unless explicitly requested |
| `GovernanceTokenSigningMaterial::build()` | Not `canonicalize()` |
| ContextPulse field order | `pulse_id\|context_id\|device_id\|session_id\|site_id\|network_id\|trust_score_4dp\|trust_level\|behavior_flags_json\|geo_zone\|network_effective_type\|session_duration\|issued_at\|expires` |
| CI auth fail-fast | Empty token must abort the step |
| AuditLedger genesis hash | `str_repeat('0', 64)` — not an empty string |
| `SieveKernel::boot()` | Must call `did_action('muplugins_loaded')` before deferring |
| GovernanceToken TTL floor | Must enforce `Platform::GOVERNANCE_TOKEN_TTL_MIN_SECONDS` |
| AES-256-GCM | Not CBC for authenticated encryption |
| No `identity_id` in ContextPulse | Replay-attack surface |
| `behavior_flags` encoding | Sorted JSON array, not CSV |

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| `ANTHROPIC_API_KEY` secret | Add to the **calling** repository's *Settings → Secrets and variables → Actions* |
| GitHub Actions enabled | The workflow uses `GITHUB_TOKEN` automatically — no extra setup needed |

---

## Usage

### Option A — Call this as a reusable workflow (recommended)

Create `.github/workflows/claude-pr-review.yml` in your repository:

```yaml
name: Claude PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  review:
    uses: AiWA-Ai-West-Africa/sparxstar-workflows/.github/workflows/claude-pr-review.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

That is all that is needed. Every time a PR is opened, updated, or reopened the review will run automatically and post a comment.

### Option B — Copy the workflow directly

If you need to customise the rules or model, copy `.github/workflows/claude-pr-review.yml` from this repository into your own and edit it as required.

---

## How the review comment looks

```
## Claude PR Review

REPOSITORY: your-org/your-repo
PR: #42 – Add payment gateway integration

VIOLATIONS (must fix before merge):
[HIGH] Missing declare(strict_types=1) in src/Payment/Gateway.php. Rule: strict_types requirement.
[CRITICAL] AES-256-CBC used in TokenEncryptor::encrypt(). Rule: AES-256-GCM not CBC.

WARNINGS (should fix):
[MEDIUM] error_log() call on line 87 — replace with platform logger.

VERDICT: FAIL

---
*Reviewed by Claude claude-sonnet-4-20250514 · [SPARXSTAR Platform Standards](https://github.com/Starisian-Technologies)*
```

> ⚠️ When a PR diff exceeds 80 KB the workflow truncates it and adds a note to the comment. Consider splitting very large PRs into smaller ones.

---

## Repository context files

The workflow automatically picks up the following files when they exist in the repository being reviewed:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Agent and AI collaboration instructions |
| `.github/copilot-instructions.md` | Copilot coding rules |
| `docs/*.md` | Architecture and design docs |
| `specs/*.md` | Platform specification files |
| `*.md` (root, excluding `README.md` / `CHANGELOG.md`) | Any other top-level Markdown context |

---

## Contributing

Pull requests and issues are welcome. When submitting a PR to this repository the Claude review workflow runs on itself, so all platform rules apply.
