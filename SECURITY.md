# Security Policy

## Supported versions

This repository is a documentation / prompt artifact (an adversarial review
harness) — there is no runtime service or shipped binary. Security-relevant
fixes, when applicable, are made against the latest release.

| Version | Supported |
| ------- | --------- |
| 1.0.x   | ✅        |
| < 1.0   | ❌        |

## Reporting a vulnerability

**Please do not open a public issue, pull request, or discussion for a security
problem** — that would disclose it before a fix is available.

Report privately through GitHub's **private vulnerability reporting**:

1. Open the repository's **[Security](https://github.com/ensell-yes/moriarty-adversarial-review/security)** tab.
2. Click **Report a vulnerability**.
3. Include a description, reproduction steps, and the impact you observed.

You will get an acknowledgement within **5 business days**, and a resolution or
mitigation timeline once the report is triaged. Please allow a reasonable window
to address the issue before any public disclosure; we will credit reporters who
ask to be credited.

## Scope

Because this repo ships Markdown rather than executable code, the most relevant
reports include:

- guidance in the harness that could steer a reviewing agent toward unsafe or
  destructive actions,
- malicious or misleading content introduced via a pull request, and
- supply-chain issues in any CI workflow (e.g. `.github/workflows/`) added to the
  repository.

Findings in your own downstream use of the harness (your tools, your data, your
pipelines) are outside this repository's scope.
