# Contributing

How we work in every **APPCO-PTE-LTD** repository. A repository's own `CONTRIBUTING.md` replaces
this one. Members will find the full rules, and the reasons behind them, in the
[SDLC documentation](https://github.com/APPCO-PTE-LTD/.github-private/tree/main/SDLC)
(§5 Phases 6–7, §10).

## Before you start

- Every change starts from an issue. Make sure it meets the Definition of Ready, and ask in the
  issue if anything is unclear.
- Assign yourself and set it to **In Progress**. One item in progress at a time.

## Branches

Branch from `nightly` and name it `<type>/<issue-number>-<short-description>`, for example
`feature/142-subscription-plan-change`. Use lowercase, hyphens, and keep it under about 60 characters.

| Prefix | For |
|---|---|
| `feature/` | New functionality |
| `fix/` | Bug fix (non-urgent) |
| `hotfix/` | Urgent production fix, branched from **`main`**, then back-merged into `nightly` within 24 h |
| `chore/` | Maintenance, dependencies, config |
| `docs/` | Documentation only |
| `refactor/` | Restructuring without behaviour change |
| `test/` | Test-only additions |

## Commits

[Conventional Commits](https://www.conventionalcommits.org/):
`<type>(<optional scope>): <short imperative description>`. Keep the subject at 72 characters or
fewer, with no trailing period. Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`,
`test`, `build`, `ci`, `chore`, `revert`. Mark an incompatible change with `!` or a
`BREAKING CHANGE:` footer.

## Pull requests

- Target `nightly`, never `main`. Use a draft PR while you are not ready for review.
- Write the title in Conventional Commits format, because it becomes the squash commit.
- Link the issue with `Closes #142`.
- Fill in every section of the template. Write "None" rather than deleting a section.
- Run the repository's local check command before pushing. Don't request review while CI is red.
- Keep it small, ideally under 400 changed lines. Split anything bigger, or explain why you can't.
- New logic ships with tests. A bug fix ships with a test that fails without the fix.
- A new environment variable goes in `.env.example` and the deployment notes, in the same PR.

## Review

- A standard change needs **1 approval**. Auth, payments, migrations, infrastructure, shared
  libraries and CI/CD need **2 approvals**, one from the Tech Lead or CODEOWNERS.
- Reviewers give a first response within one working day.
- Prefix non-blocking comments with `nit:` or `question:`.
- Answer every comment. Push fixes as new commits, with no force-pushing mid-review.
- Resolve all conversations before merging, then **squash and merge** and trim the squash message.

## After merge

Merging to `nightly` deploys to Preview. Check that your change works there, then move the issue to
**Ready for QA**.

## Security

Never report a vulnerability in a public issue. See [SECURITY.md](SECURITY.md).
