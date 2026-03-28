# SPEC.md — us-code Specification

## 1. Overview

**us-code** represents the entire United States Code, its legislative history, and the ongoing legislative process as a Git repository. Every section of federal law is a markdown file. Every enacted law is a merged pull request. Every bill is tracked as a PR through its lifecycle, with vote records, party affiliations, and member data attached.

The repository serves three purposes:
1. **Historical record** — browse the evolution of US law via `git log`, `git diff`, and `git blame`
2. **Living tracker** — open PRs represent pending legislation with real-time status updates
3. **Simulation platform** — draft PRs enable AI-driven legislative debate and vote prediction

## 2. Content Model

### 2.1 US Code Sections

Each section of the United States Code is a single markdown file.

**Path pattern:** `uscode/title-{NN}/section-{NNN}.md`

**Example:** `uscode/title-47/section-230.md`

```markdown
---
title: 47
section: 230
heading: "Protection for private blocking and screening of offensive material"
enacted: "1996-02-08"
public_law: "PL 104-104"
last_amended: "1998-10-21"
last_amended_by: "PL 105-277"
status: "in-force"
source: "https://uscode.house.gov/view.xhtml?req=granuleid:USC-prelim-title47-section230"
---

# § 230. Protection for private blocking and screening of offensive material

## (a) Findings

The Congress finds the following:

(1) The rapidly developing array of Internet and other interactive computer services...
```

**Frontmatter fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | number | yes | USC title number |
| `section` | string | yes | Section number (may include letters, e.g., "36B") |
| `heading` | string | yes | Official section heading |
| `enacted` | date | yes | Date the section was first enacted |
| `public_law` | string | yes | Public law that created this section |
| `last_amended` | date | no | Date of most recent amendment |
| `last_amended_by` | string | no | Public law that last amended this section |
| `status` | enum | yes | `in-force`, `repealed`, `transferred`, `omitted` |
| `source` | url | yes | Link to official OLRC source |

**Title metadata file:** Each title directory contains a `_title.md` with:
```markdown
---
title: 47
heading: "Telecommunications"
positive_law: true
chapters: 16
sections: 614
---

# Title 47 — Telecommunications

Enacted as positive law: Yes
```

### 2.2 Constitution

**Path pattern:** `constitution/article-{N}.md`, `constitution/amendment-{NN}.md`

```markdown
---
type: "amendment"
number: 1
ratified: "1791-12-15"
proposed: "1789-09-25"
proposing_congress: "1st"
---

# Amendment I

Congress shall make no law respecting an establishment of religion, or prohibiting
the free exercise thereof; or abridging the freedom of speech, or of the press; or
the right of the people peaceably to assemble, and to petition the Government for a
redress of grievances.
```

### 2.3 Public Law Records

**Path pattern:** `public-laws/{congress}/PL-{congress}-{number}.md`

```markdown
---
public_law: "PL 116-136"
congress: 116
bill: "HR 748"
title: "Coronavirus Aid, Relief, and Economic Security Act"
short_title: "CARES Act"
sponsor: "Rep. Joe Courtney (D-CT-2)"
enacted: "2020-03-27"
signed_by: "President Donald J. Trump"
sections_amended: ["26-36B", "26-72", "26-127", "42-247d", "15-9001"]
sections_created: ["15-9001", "15-9002", "15-9003"]
sections_repealed: []
source: "https://www.congress.gov/bill/116th-congress/house-bill/748"
govinfo: "https://www.govinfo.gov/app/details/PLAW-116publ136"
---

# PL 116-136 — CARES Act

Coronavirus Aid, Relief, and Economic Security Act

## Summary
...

## Vote Record
See PR #116-136 for full roll call votes.
```

### 2.4 Member Profiles

**Path pattern:** `members/{chamber}/{state}-{last_name}-{first_name}.md`

```markdown
---
name: "Alexandria Ocasio-Cortez"
bioguide_id: "O000172"
chamber: "house"
state: "NY"
district: 14
party: "Democrat"
terms:
  - congress: 116
    start: "2019-01-03"
    end: "2021-01-03"
  - congress: 117
    start: "2021-01-03"
    end: "2023-01-03"
committees: ["Financial Services", "Oversight and Reform"]
ideology_score: -0.523  # DW-NOMINATE (VoteView)
source: "https://bioguide.congress.gov/search/bio/O000172"
---
```

### 2.5 Vote Records

**Path pattern:** `votes/{congress}/{chamber}/roll-call-{number}.md`

```markdown
---
congress: 116
chamber: "senate"
roll_call: 80
date: "2020-03-25"
bill: "HR 748"
question: "On Passage of the Bill"
result: "Passed"
yea: 96
nay: 0
not_voting: 4
required: "3/5"
source: "https://www.senate.gov/legislative/LIS/roll_call_votes/vote1162/vote_116_2_00080.htm"
---

# Roll Call Vote 80 — 116th Congress, 2nd Session

**Question:** On Passage of the Bill (H.R. 748 as Amended)
**Result:** Passed, 96-0

| Party       | Yea | Nay | Not Voting |
|-------------|-----|-----|------------|
| Republican  | 52  | 0   | 1          |
| Democrat    | 44  | 0   | 3          |

## Individual Votes

### Yea (96)
- Alexander (R-TN)
- Baldwin (D-WI)
...

### Not Voting (4)
- Markey (D-MA)
...
```

## 3. Git Model

### 3.1 Branching

```
main                  # Living US Code — only enacted law
bills/hr-{number}     # House bill branches
bills/s-{number}      # Senate bill branches
bills/hjres-{number}  # House joint resolutions
bills/sjres-{number}  # Senate joint resolutions
```

**Rules:**
- `main` is protected. Only the sync engine merges into it.
- Bill branches are created when a bill is introduced.
- Bill branches contain the proposed changes to `uscode/` section files.
- Amendments to a bill = additional commits on the branch.

### 3.2 Commits

**Enacted law commit:**
```
Author: Rep. Joe Courtney <courtney@congress.gov>
AuthorDate: Fri Mar 27 00:00:00 2020 -0500
Commit: us-code-sync <sync@us-code>
CommitDate: <actual ingestion timestamp>

PL 116-136: Coronavirus Aid, Relief, and Economic Security Act

Sponsor: Rep. Joe Courtney (D-CT-2)
Cosponsors: 0
Signed: 2020-03-27 by President Donald J. Trump

House: PASSED 419-6 (2020-03-27)
Senate: PASSED 96-0 (2020-03-25)

Source: https://www.congress.gov/bill/116th-congress/house-bill/748
```

**Author date** = enactment date (for chronological `git log`).
**Commit date** = actual ingestion time (for audit trail).
**Author** = bill sponsor (for `git blame` attribution).

### 3.3 Pull Requests

**PR number convention:** `{congress}-{law_number}` where possible (e.g., PR title "PL 116-136"). For pending bills, use sequential numbering.

**PR body template:**
```markdown
# HR 1234 — [Bill Title]
[Congress]th Congress ([year range])

## Status: [CURRENT_STATUS]
Chamber: [House|Senate]
Committee: [Committee Name]

## Sponsor
[Name] ([Party]-[State]-[District]) | Introduced: [Date]

## Cosponsors ([count])
[List]

## Timeline
| Date | Action |
|------|--------|
| ...  | ...    |

## Votes
[Vote tables when available]

## Sections Affected
- `uscode/title-XX/section-YYY.md` — [description of change]
```

**PR lifecycle:**
1. Bill introduced → PR opened (status: `introduced`)
2. Committee referral → PR comment + label update
3. Committee action → PR comment (hearings, markup, amendments as commits)
4. Floor vote → vote record attached, PR comment
5. Second chamber → PR comment tracking parallel progress
6. Conference → PR comment
7. Enrolled → PR comment
8. Signed → PR merged with backdated commit
9. Vetoed / Died → PR closed with final status label

### 3.4 Labels

Fixed, controlled set:

**Status labels** (mutually exclusive):
- `introduced` — bill has been introduced
- `in-committee` — referred to committee
- `passed-house` — passed House floor vote
- `passed-senate` — passed Senate floor vote
- `passed-both` — passed both chambers (awaiting signature)
- `signed` — signed into law
- `vetoed` — vetoed by President
- `veto-overridden` — veto overridden by Congress
- `died` — died in committee or tabled
- `expired` — Congress ended without action

**Chamber labels:**
- `house`, `senate`, `joint`

**Congress labels:**
- `1st` through `119th` (and beyond)

**Category labels** (non-exclusive):
- `appropriations`, `defense`, `healthcare`, `tax`, `judiciary`, `civil-rights`, `environment`, `education`, `immigration`, `trade`, `technology`, `infrastructure`, `agriculture`, `financial-regulation`

## 4. Data Sources & Ingestion

### 4.1 Source Priority

| Priority | Source | Data | Format | Auth |
|----------|--------|------|--------|------|
| 1 | OLRC (uscode.house.gov) | Consolidated US Code | USLM XML | None |
| 2 | GovInfo API | Enrolled bills, public laws, Statutes at Large | XML/JSON | api.data.gov key |
| 3 | Congress.gov API | Bills, amendments, members, votes | JSON | api.data.gov key |
| 4 | VoteView | Historical roll calls, ideology scores | CSV | None |
| 5 | @unitedstates project | Legislator YAML, supplementary metadata | YAML/JSON | None |

### 4.2 USLM XML → Markdown Transform

The OLRC publishes the US Code in USLM XML format. The transform pipeline:

1. Download title ZIP from OLRC (one ZIP per title)
2. Parse USLM XML — extract sections with heading, content, notes
3. Convert XML structure to markdown:
   - `<section>` → file
   - `<subsection>` → `## (a)`
   - `<paragraph>` → `(1)` indented
   - `<subparagraph>` → `(A)` double-indented
   - `<note>` → blockquote or footnote
4. Generate frontmatter from XML attributes + cross-reference data
5. Write to `uscode/title-{NN}/section-{NNN}.md`

### 4.3 Sync Schedule

| Job | Frequency | Source | Action |
|-----|-----------|--------|--------|
| New bills | Daily | Congress.gov API | Create PR + branch |
| Bill status updates | Daily | Congress.gov API | Update PR body + comments |
| Newly enacted laws | Daily | GovInfo | Merge PR + update uscode/ files |
| Vote records | Daily | Congress.gov API | Attach to PR |
| US Code reconciliation | Weekly | OLRC | Verify/correct uscode/ against official |
| Member data refresh | Monthly | Congress.gov + VoteView | Update member profiles |

### 4.4 Historical Backfill

**Phase 1 — Constitution (27 commits)**
- One commit per ratification event (original + 27 amendments)
- Author dates = ratification dates (1788 → present)

**Phase 2 — Baseline US Code**
- Start from earliest available OLRC snapshot
- Single commit: "US Code as of [date]"
- This is the foundation that subsequent law-commits modify

**Phase 3 — Historical Laws (chronological)**
- Ingest public laws chronologically from earliest available
- Each law = a commit modifying the relevant section files
- Author date = enactment date
- Vote records attached where available (VoteView has roll calls back to 1789)

**Phase 4 — Modern Bills (with PRs)**
- From a chosen Congress onward (e.g., 113th / 2013), create full PR representation
- Includes failed bills, committee actions, amendments

### 4.5 Ingestion State

Track sync progress in `scripts/state/sync-state.json`:
```json
{
  "lastSync": "2026-03-28T12:00:00Z",
  "bills": {
    "lastCongress": 119,
    "lastBillIngested": "HR-1234",
    "lastStatusCheck": "2026-03-28T12:00:00Z"
  },
  "laws": {
    "lastPublicLaw": "PL-119-12",
    "lastEnactmentDate": "2026-02-15"
  },
  "uscode": {
    "lastOlrcRefresh": "2026-03-25",
    "titlesRefreshed": [1, 2, 3, "..."]
  },
  "votes": {
    "lastRollCall": { "senate": 45, "house": 102 }
  }
}
```

## 5. Tooling

### 5.1 Tech Stack

- **Runtime:** Node.js (TypeScript)
- **XML parsing:** `fast-xml-parser`
- **HTTP:** Native `fetch` (Node 22+)
- **Git operations:** `simple-git` or direct `exec` of git CLI
- **GitHub API:** `@octokit/rest`
- **Markdown:** `gray-matter` (frontmatter), custom USLM transformer
- **Scheduling:** Node cron or external (GitHub Actions / system cron)
- **Testing:** Vitest

### 5.2 Project Structure

```
scripts/
├── src/
│   ├── index.ts                # CLI entry point
│   ├── sync/
│   │   ├── orchestrator.ts     # Main sync loop
│   │   ├── scheduler.ts        # Cron scheduling
│   │   └── state.ts            # Ingestion state management
│   ├── sources/
│   │   ├── olrc.ts             # OLRC US Code download
│   │   ├── govinfo.ts          # GovInfo API client
│   │   ├── congress.ts         # Congress.gov API client
│   │   ├── voteview.ts         # VoteView CSV parser
│   │   └── unitedstates.ts     # @unitedstates data loader
│   ├── transforms/
│   │   ├── uslm-to-markdown.ts # USLM XML → section markdown
│   │   ├── bill-to-pr.ts       # Bill data → PR body
│   │   ├── vote-formatter.ts   # Roll call → markdown table
│   │   └── member-profile.ts   # Member data → markdown profile
│   ├── git/
│   │   ├── commit.ts           # Create backdated commits
│   │   ├── branch.ts           # Bill branch management
│   │   └── pr.ts               # GitHub PR create/update/close
│   └── utils/
│       ├── rate-limit.ts       # API rate limiting
│       ├── cache.ts            # Response caching
│       └── logger.ts           # Structured logging
├── tests/
│   ├── transforms/
│   ├── sources/
│   └── fixtures/               # Sample USLM XML, API responses
├── state/
│   └── sync-state.json
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

### 5.3 CLI Interface

```bash
# Full sync (all sources)
npx us-code sync

# Sync specific source
npx us-code sync --source=bills
npx us-code sync --source=laws
npx us-code sync --source=votes
npx us-code sync --source=uscode

# Historical backfill
npx us-code backfill --phase=constitution
npx us-code backfill --phase=baseline
npx us-code backfill --phase=laws --congress=116
npx us-code backfill --phase=bills --congress=118

# Utilities
npx us-code status              # Show sync state
npx us-code validate            # Verify uscode/ against OLRC
npx us-code transform <file>    # Convert single USLM XML → markdown
```

## 6. Simulation Platform (Phase 4)

### 6.1 Concept

Draft PRs represent hypothetical legislation. AI agents with assigned political personas debate the proposal:

- **Party agents** — configured with party platform positions, historical voting patterns, and ideology scores (DW-NOMINATE from VoteView)
- **Committee agents** — simulate committee markup and amendment process
- **Constitutional analyst** — reviews for constitutional issues and precedent
- **CBO simulator** — estimates fiscal impact based on historical patterns

### 6.2 Simulation Flow

1. User creates a draft PR with proposed law changes
2. System assigns agents based on relevant committees
3. Agents post analysis as PR review comments
4. Amendment agents propose changes (commits on the branch)
5. Floor debate agents argue for/against
6. Vote prediction based on historical patterns + agent analysis
7. Final simulated vote posted

### 6.3 Agent Configuration

```yaml
# Example party agent
name: "Conservative Republican"
ideology:
  economic: 0.7      # DW-NOMINATE dim 1
  social: 0.5        # DW-NOMINATE dim 2
priorities:
  - fiscal_responsibility
  - deregulation
  - national_defense
  - states_rights
voting_patterns:
  source: "voteview"
  filter: "party=Republican AND nominate_dim1 > 0.5"
```

## 7. Constraints & Considerations

### 7.1 Scale

- US Code: ~54 titles, ~15,000+ sections, ~200,000+ pages
- Public laws: ~300-600 per Congress, 119 Congresses = ~50,000+ laws
- Bills: ~10,000-15,000 per Congress (most fail)
- Roll call votes: ~800-1,200 per Congress
- Members: ~12,000+ historical

### 7.2 Git Performance

At scale, the repo will be large. Mitigations:
- Use shallow clones for most operations
- Consider git LFS for large reference documents (not section files)
- Partition historical data if needed (archive old Congresses)
- Keep individual files small (section-level granularity helps)

### 7.3 Rate Limits

- Congress.gov API: 5,000 requests/hour (with API key)
- GovInfo API: Similar limits
- GitHub API: 5,000 requests/hour (authenticated)
- Build in backoff, caching, and incremental sync

### 7.4 Data Quality

- OLRC consolidation lags enactment by days to weeks
- Some historical data is incomplete (pre-1970s vote records, early Congress metadata)
- Cross-reference multiple sources for accuracy
- Track confidence/completeness in frontmatter
