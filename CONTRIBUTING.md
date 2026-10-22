# Contributing

retro-plumber is currently a design-study repository (GDD, ADRs, UML). No game code exists yet.

- **Branch** from `main`: `feat/<scope>`, `fix/<scope>`, `refactor/<scope>`, `chore/<scope>`,
  `docs/<scope>`. Never commit to `main`.
- **Commits** follow Conventional Commits: `<type>(<scope>): <description>`. Scopes: `design`,
  `study`, `adr`, `uml`, `docs`, `ci`, `repo`.
- **Design before code**: a feature needs its rules in the GDD and validated UML diagrams under
  `docs/architecture/diagrams/<feature>/` first.
- **No third-party intellectual property** in code, assets, docs, commits or issues.
- **Language**: English only in the repository.
- **Checks**: Gitleaks, CodeQL and dependency review run in CI; SonarQube runs locally with
  `sonar-scanner` (the token comes from `SONAR_TOKEN`, never from a file in the repository).
- **Pull requests** use the template, link their issue (`Closes #N`) and are merged after review.
