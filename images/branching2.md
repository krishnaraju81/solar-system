# Release Train Branching Strategies
## GitFlow-based vs Scaled TBD · Side-by-side Comparison

---

## Strategy 1 · GitFlow-based Release Train

```mermaid
%%{init: { 'theme': 'base', 'gitGraph': {'rotateCommitLabel': false}} }%%
gitGraph LR:
   commit id: "v1.0.0" tag: "v1.0.0"

   branch develop
   checkout develop
   commit id: "develop baseline"

   branch feature/payment-retry
   checkout feature/payment-retry
   commit id: "feat: payment retry"

   checkout develop
   merge feature/payment-retry id: "✓ → dev ns"

   branch project/payments-overhaul
   checkout project/payments-overhaul
   commit id: "project: initial structure"

   checkout develop
   branch feature/search-filters
   checkout feature/search-filters
   commit id: "feat: search filters"

   checkout develop
   merge feature/search-filters id: "✓ → dev ns "

   checkout project/payments-overhaul
   merge develop id: "⟳ sync from develop"
   commit id: "project: core domain"

   checkout develop
   branch release/v1.1.0
   checkout release/v1.1.0
   commit id: "cut from develop · → staging"
   commit id: "fix: edge case"

   checkout main
   merge release/v1.1.0 id: "✓ regression → [gate]" tag: "v1.1.0 ──► prod"

   checkout develop
   merge release/v1.1.0 id: "back-merge v1.1.0 → develop"

   branch feature/user-profile
   checkout feature/user-profile
   commit id: "feat: user profile"

   checkout develop
   merge feature/user-profile id: "✓ → dev ns  "

   checkout project/payments-overhaul
   merge develop id: "⟳ sync from develop "
   commit id: "project: complete"

   checkout develop
   merge project/payments-overhaul id: "✓ project merged → dev ns"

   branch release/v2.0.0
   checkout release/v2.0.0
   commit id: "cut from develop · MAJOR · → staging"
   commit id: "fix: regression"

   checkout main
   merge release/v2.0.0 id: "✓ regression → [gate] " tag: "v2.0.0 ──► prod"

   checkout develop
   merge release/v2.0.0 id: "back-merge v2.0.0 → develop"

   branch hotfix/v2.0.1
   checkout hotfix/v2.0.1
   commit id: "fix: critical prod bug"

   checkout main
   merge hotfix/v2.0.1 id: "✓ smoke → [gate]  " tag: "v2.0.1 ──► prod"

   checkout develop
   merge hotfix/v2.0.1 id: "back-merge hotfix → develop"
```

### Key observations
- `develop` acts as a crash buffer — broken commits never touch `main`
- Release branch is cut **from `develop`**, not `main`
- Back-merges go to **both** `main` and `develop` after every release and hotfix
- Project branch syncs **from `develop`** — may occasionally sync unstable code
- Tag lives on `main` after the release branch merges in
- Teams working on features are unaffected by release stabilisation (on `develop`)

---

## Strategy 2 · Scaled TBD Release Train

```mermaid
%%{init: { 'theme': 'base', 'gitGraph': {'rotateCommitLabel': false}} }%%
gitGraph LR:
   commit id: "v1.0.0" tag: "v1.0.0"

   branch feature/payment-retry
   checkout feature/payment-retry
   commit id: "feat: payment retry"

   branch project/payments-overhaul
   checkout project/payments-overhaul
   commit id: "project: initial structure"

   checkout main
   merge feature/payment-retry id: "✓ → dev ns"

   branch feature/search-filters
   checkout feature/search-filters
   commit id: "feat: search filters"

   checkout main
   merge feature/search-filters id: "✓ → dev ns "

   checkout project/payments-overhaul
   merge main id: "⟳ sync from main"
   commit id: "project: core domain"

   checkout main
   branch release/v1.1.0
   checkout release/v1.1.0
   commit id: "cut from main · → staging"
   commit id: "fix: edge case" tag: "v1.1.0 ──► prod"

   checkout main
   commit id: "cherry-pick: edge case fix"

   branch feature/user-profile
   checkout feature/user-profile
   commit id: "feat: user profile"

   checkout main
   merge feature/user-profile id: "✓ → dev ns  "

   checkout project/payments-overhaul
   merge main id: "⟳ sync from main "
   commit id: "project: complete"

   checkout main
   merge project/payments-overhaul id: "✓ project merged → dev ns"

   branch release/v2.0.0
   checkout release/v2.0.0
   commit id: "cut from main · MAJOR · → staging"
   commit id: "fix: regression" tag: "v2.0.0 ──► prod"

   checkout main
   commit id: "cherry-pick: regression fix"

   branch hotfix/v2.0.1
   checkout hotfix/v2.0.1
   commit id: "fix: critical prod bug"

   checkout main
   merge hotfix/v2.0.1 id: "✓ smoke → [gate]" tag: "v2.0.1 ──► prod"
```

### Key observations
- No `develop` — `main` is the only integration point and must always be green
- Release branch cut **from `main`** directly
- Stabilisation fixes go onto `release/*` first, then **cherry-picked back to `main`** — never a full back-merge
- Tag lives **on the release branch**, not on `main`
- Project branch syncs **from `main`** — always syncing against production-quality code
- Teams keep merging to `main` while the release branch stabilises (visible after v1.1.0 cut)
- Hotfix merges directly to `main` and gets tagged there

---

## Head-to-head comparison

| | GitFlow-based Release Train | Scaled TBD Release Train |
|---|---|---|
| Branch count | 6 types | 5 types (no `develop`) |
| Integration point | `develop` | `main` |
| Dev namespace deploys from | `develop` | `main` |
| Release branch cut from | `develop` | `main` |
| Tag lives on | `main` | `release/*` branch |
| Stabilisation fix direction | `release/*` → back-merge to `main` + `develop` | `release/*` → cherry-pick to `main` |
| Project branch syncs from | `develop` (may be unstable) | `main` (always production-quality) |
| Broken commit consequence | `develop` breaks — `main` safe | `main` breaks — whole team stops |
| Hotfix back-merge | `main` + `develop` | `main` only |
| Discipline required | Medium | High |
| Best for | Teams building CI discipline | Teams with strong test coverage and fast pipelines |

---

## Feature freeze rule (both strategies)

PRs from `project/*` to `develop` (GitFlow) or `main` (Scaled TBD) must be merged
**at least 2 working days before the release branch is cut** — no exceptions.
Miss the window and the project catches the next train.

```
3-week minor  →  feature freeze on day 16 of 21  →  release branch cut day 18
6-week major  →  feature freeze on day 33 of 42  →  release branch cut day 36
```
