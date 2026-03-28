# United States Code

The United States Code as a Git repository.

Every enacted law is a merged pull request. Every failed bill is a closed pull request. Every vote is on record.

The `main` branch is the living United States Code — the current state of US federal statutory law. Git history is American legal history.

```bash
# What does Section 230 say?
cat uscode/title-47/section-230.md

# When was it last amended?
git log --oneline -- uscode/title-47/section-230.md

# What exactly changed?
git diff <commit>^ <commit> -- uscode/title-47/section-230.md
```

## Structure

```
constitution/          # Articles I-VII + 27 Amendments
uscode/                # 54 titles, one file per section
  title-01/
  title-02/
  ...
public-laws/           # Enacted law metadata by Congress
members/               # Legislator profiles
votes/                 # Roll call vote records
```

## How It Works

- **`main`** = current US law (only enacted legislation is merged here)
- **Open PRs** = pending legislation tracking through Congress
- **Merged PRs** = enacted laws with full vote records and history
- **Closed PRs** = failed or expired bills
- **`git blame`** = trace any section to the law that wrote it
- **`git log`** = chronological history of American law

## Data Sources

All content is public domain (US government works):

- [Office of the Law Revision Counsel](https://uscode.house.gov) — US Code (USLM XML)
- [GovInfo](https://govinfo.gov) — Public laws and Statutes at Large
- [Congress.gov](https://api.congress.gov) — Bills, amendments, members, votes
- [VoteView](https://voteview.com) — Historical roll calls and ideology scores

## Tooling

Ingestion and sync tooling lives in a separate repository: [us-code-tools](https://github.com/v1d0b0t/us-code-tools)

## License

Legislative content: **Public domain** (US government works).
Repository structure and metadata: **MIT**.
