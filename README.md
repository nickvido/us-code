# us-code

**United States Code as a Git repository.**

Every enacted law is a merged PR. Every failed bill is a closed PR. Every vote is on record.

The `main` branch is the living United States Code — the current state of US federal statutory law. Git history is American legal history: `git log` shows every law ever enacted, `git diff` shows exactly what changed, and `git blame` traces any section back to the law that wrote it.

## How It Works

- **One file per section** of the US Code (~15,000+ sections across 54 titles)
- **Each enacted law** = a merged pull request with backdated commit
- **Each pending bill** = an open pull request tracking its lifecycle
- **Each failed bill** = a closed pull request with vote records and debate history
- **Vote records** on every PR — roll call tallies, party breakdowns, individual member votes
- **Member profiles** with party affiliation, committee assignments, voting history

## Repository Structure

```
uscode/
├── title-01/                  # General Provisions
│   ├── _title.md              # Title metadata
│   ├── section-1.md           # §1. Words denoting number, gender, and so forth
│   ├── section-2.md
│   └── ...
├── title-18/                  # Crimes and Criminal Procedure
├── title-26/                  # Internal Revenue Code
├── title-42/                  # The Public Health and Welfare
└── ...                        # 54 titles total

constitution/
├── article-I.md
├── article-II.md
├── ...
├── amendment-01.md            # Bill of Rights
├── amendment-02.md
└── amendment-27.md

public-laws/                   # Enacted law metadata & full text
├── 001/                       # 1st Congress (1789-1791)
├── ...
└── 119/                       # Current Congress

members/                       # Legislator profiles
├── senate/
└── house/

votes/                         # Roll call vote records
├── 001/
└── ...

scripts/                       # Ingestion & sync tooling (TypeScript)
├── src/
│   ├── sync/                  # Orchestrator + scheduling
│   ├── sources/               # API clients (Congress.gov, GovInfo, OLRC, VoteView)
│   ├── transforms/            # USLM XML → markdown, vote formatting
│   └── git/                   # Commit, branch, PR management
└── ...

docs/
├── SPEC.md                    # Full specification
└── ARCHITECTURE.md
```

## The Git Model

### `main` = Current US Law

The `main` branch always reflects the current consolidated United States Code. Only enacted, signed laws are merged into `main`.

### Bills as Pull Requests

Every bill introduced in Congress becomes a PR:

- **Branch:** `bills/hr-1234` or `bills/s-456`
- **PR body:** Bill metadata, sponsor, timeline, current status
- **PR comments:** Status updates as the bill progresses (committee hearings, markup, floor votes)
- **Merge:** When signed into law, PR merges with the signing date as commit author date
- **Close:** When a bill dies (expires, voted down, vetoed), PR closes with final status

### Labels (Fixed Set)

```
Status:    introduced | in-committee | passed-house | passed-senate | passed-both | signed | vetoed | died | expired
Chamber:   house | senate | joint
Congress:  118th | 119th | ...
Category:  appropriations | defense | healthcare | tax | judiciary | ...
```

### Vote Records

Every floor vote is captured on the PR:

```markdown
## House Vote — 2023-09-20
Result: PASSED 267-158

| Party       | Yea | Nay | Not Voting |
|-------------|-----|-----|------------|
| Republican  | 55  | 157 | 6          |
| Democrat    | 212 | 1   | 0          |

<details><summary>Individual votes</summary>
Rep. Jane Smith (D-CA-12): Yea
Rep. John Doe (R-TX-07): Nay
...
</details>
```

## Data Sources

All source data is public domain (US government works):

| Source | Data | URL |
|--------|------|-----|
| Office of the Law Revision Counsel | US Code (USLM XML) | uscode.house.gov |
| GovInfo (GPO) | Enrolled bills, public laws, Statutes at Large | govinfo.gov |
| Congress.gov API | Bills, amendments, members, votes | api.congress.gov |
| VoteView | Historical roll calls (back to 1789), ideology scores | voteview.com |
| @unitedstates project | Legislator data, bill metadata | github.com/unitedstates |

## Future: Simulation Platform

Propose hypothetical laws as draft PRs and simulate Congressional debate:

- Party-affiliated AI agents analyze and debate proposed changes
- Agents score proposals against historical voting patterns and party platforms
- Simulated committee markup, floor debate, and vote predictions
- Constitutional analysis and precedent review

## Getting Started

```bash
git clone git@github.com:v1d0b0t/us-code.git
cd us-code

# What does Section 230 say today?
cat uscode/title-47/section-230.md

# When was it last amended?
git log --oneline -- uscode/title-47/section-230.md

# What did the CARES Act change?
gh pr view 116-136

# How did your representative vote?
grep "Jane Smith" votes/116/house/roll-call-102.md
```

## License

Legislative content: **Public domain** (US government works).
Repository structure, tooling, and metadata: **MIT**.
