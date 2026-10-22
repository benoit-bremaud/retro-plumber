# Security Policy

## Supported versions

Only the current state of `main` receives security fixes.

## Reporting a vulnerability

Please do not open a public issue for a security problem. Use GitHub's private vulnerability
reporting: open the **Security** tab of the repository and click **Report a vulnerability**.
Include a description, steps to reproduce and the commit where you observed it. You will receive
an acknowledgment within **72 hours**.

## Security practices

- **Secret detection** — Gitleaks runs in CI on every push and pull request, plus weekly.
- **Static analysis** — CodeQL (workflows now, game code once it exists), SonarQube locally.
- **Pinned actions** — every GitHub Actions step is pinned to a full commit SHA.
- **Dependency monitoring** — Dependabot and dependency review.
- **No backend, no personal data** — the game is planned as a static web page with no server,
  account or network call.
