# Branch strategy

ThreadWise is built in four **cumulative** weekly branches. Each week starts from the accepted end of the previous week, so every branch contains all earlier work.

```text
main ──► week-01 ──► week-02 ──► week-03 ──► week-04
```

This follows the Version 7 delivery plan ("Branch and learner workflow") and the milestones in [backlog.md](./backlog.md).

## Branches

| Branch | Created from | Purpose | Backlog |
|---|---|---|---|
| `main` | — | Stable project guide: README, `docs/`, configuration templates. Contains no weekly application work. | — |
| `week-01` | `main` | Foundation: environment, packs, schema, basic Learn mode | `W1-*` |
| `week-02` | accepted tip of `week-01` | Retrieval, MCP, planning skill, Practice | `W2-*` |
| `week-03` | accepted tip of `week-02` | Agents, Interview, plans, memory, approvals, events, authentication, AWS pilot | `W3-*` |
| `week-04` | accepted tip of `week-03` | Freeze, evaluation, repair, release (no new modes or features) | `W4-*` |

Preparation items (`PREP-*`) are entry criteria and are completed on `main` or outside the repository, before `week-01` starts.

## Rules

1. **Create a week branch only after the previous week is accepted.** Do not start `week-02` work until the `week-01` review gate in the backlog is met.
2. **Review each branch against its predecessor.** Open the pull request for `week-02` with base `week-01`, for `week-03` with base `week-02`, and so on. `week-01` is reviewed against `main`.
3. **Tag every accepted milestone** with an immutable tag: `cohort-01-week-01-v1`, `cohort-01-week-02-v1`, and so on. Never move or delete a published tag.
4. **Fix forward.** Do not rewrite history or change a learner's baseline. Fix a defect on the earliest affected branch, then merge that branch forward through the later ones and publish a new tag version (for example `cohort-01-week-02-v2`).
5. **`main` changes only through reviewed pull requests.** Documentation updates made on `main` are merged forward into the open week branches.
6. **Deploy only a reviewed release.** Production deployment uses an accepted tag from `week-04`, never a branch head.
7. **No force-push** to `main` or any week branch once it has been published.
8. **Week 4 adds no features.** It is limited to evaluation, repair and release.
9. **Never commit secrets, local data, generated embeddings or holdout evaluation material.** See [.gitignore](../.gitignore) and requirement ISS-07.
10. Reference the backlog ID in each commit message or pull request (for example `W1-04: chunk passages within 256 word pieces`).

Whether `week-04` is merged back into `main` when the project is released is a stakeholder decision that is not defined in the source documents. Record it in [requirements.md](./requirements.md) before release.

## Setting up the branches

Run these once, after the repository exists on GitHub and `main` contains the bootstrap commit.

```bash
git switch main
git pull --ff-only
git switch -c week-01
git push -u origin week-01
```

When `week-01` is accepted and tagged:

```bash
git switch week-01
git pull --ff-only
git tag -a cohort-01-week-01-v1 -m "Week 01 accepted"
git push origin cohort-01-week-01-v1
git switch -c week-02
git push -u origin week-02
```

Repeat the same pattern for `week-03` (from `week-02`) and `week-04` (from `week-03`).

### Pull requests

```bash
gh pr create --base main --head week-01 --title "week-01: Foundation"
gh pr create --base week-01 --head week-02 --title "week-02: Retrieval and practice"
gh pr create --base week-02 --head week-03 --title "week-03: Agents and interview"
gh pr create --base week-03 --head week-04 --title "week-04: Release"
```

Open each pull request when the branch is ready for review. These are review pull requests, so keep the predecessor branch intact after merging.

## Fixing an earlier week

Example: a defect is found in `week-01` code while on `week-03`.

```bash
git switch week-01
git pull --ff-only
# make and commit the fix, referencing the backlog ID
git push
git tag -a cohort-01-week-01-v2 -m "Fix <short description>"
git push origin cohort-01-week-01-v2

git switch week-02 && git merge week-01 && git push
git tag -a cohort-01-week-02-v2 -m "Includes week-01 fix"
git push origin cohort-01-week-02-v2

git switch week-03 && git merge week-02 && git push
```

Continue through the remaining branches. If the fix changes behaviour that was evaluated, record that the evaluation needs a new version (see requirements section 11.5).

## Pulling documentation changes forward

```bash
git switch week-01 && git merge main
git switch week-02 && git merge week-01
git switch week-03 && git merge week-02
git switch week-04 && git merge week-03
```

Only merge into branches that already exist.

## Working in a Codespace

Use one Codespace across the weekly branches. **Commit or stash before switching branches.** Moving backwards may require the database reset procedure and migrations for that checkpoint; the reset commands are added in backlog task `W1-01`. See [codespaces.md](./codespaces.md).

## Optional repository protection

Branch protection (required reviews, no force-push, status checks) is not configured by this repository. If you enable it, protect `main` and the week branches and require the CI check once it exists (`W1-11`). Protection settings may depend on your GitHub plan and repository visibility.
