# Creating and rebuilding a GitHub Codespace

ThreadWise is developed in GitHub Codespaces. This guide covers creating, configuring, rebuilding, stopping and deleting a Codespace for this repository.

> **Status:** the repository does not yet contain a `.devcontainer/` definition. Until backlog task `W1-01` adds one, a new Codespace uses GitHub's default development image: it has no pinned Python version and no PostgreSQL/pgvector container. This guide stays valid; the rebuild step then reads the repository's dev container once it exists.

The repository is `BITS-Pilani-Digital/agentic_ai_capstone`. In the commands below, replace `OWNER/REPO` with that name.

Because the repository belongs to an organisation, Codespaces usage may be billed to the organisation or to your account, depending on the organisation's settings, and organisation-funded Codespaces has no included free quota. Confirm who pays before creating a Codespace. At the time of writing, the available machine types for this repository are `basicLinux32gb` (2 cores, 8 GB) and `standardLinux32gb` (4 cores, 16 GB).

## 1. Before you start

- You need a GitHub account with access to the repository and an active Codespaces entitlement.
- Included usage depends on the account type. The Version 7 plan cites 120 core-hours and 15 GB-month of storage for Free personal accounts and 180 core-hours and 20 GB-month for Pro; organisation-funded Codespaces has no included free quota. A two-core machine consumes two core-hours per hour. Confirm current values in GitHub's Codespaces billing documentation and in your account's billing settings.
- This project assumes **no paid resources**. Check your Codespaces spending limit or budget so usage cannot exceed the included allowance unless you have explicitly agreed to pay.
- Free inference does not mean unlimited development compute. Stop idle Codespaces and delete unused ones.

## 2. Create a Codespace

### In the browser

1. Open the repository on GitHub and choose the branch you will work on (for example `week-01`).
2. Select **Code**, then the **Codespaces** tab, then **Create codespace on `<branch>`**.
3. For options such as machine type, use **... → New with options** instead.

### With the GitHub CLI

```bash
gh auth status
gh codespace create -R OWNER/REPO -b week-01 \
  -d "threadwise-week-01" \
  --idle-timeout 30m \
  --retention-period 72h
```

- Add `-m <machine>` to choose a machine type. List available types with:

  ```bash
  gh api repos/OWNER/REPO/codespaces/machines
  ```

- Start with the smallest machine that works and increase only when measured memory requires it (backlog `PREP-08`, `W1-12`).
- To use a specific dev container file once several exist: `--devcontainer-path .devcontainer/devcontainer.json`.
- Open it afterwards:

  ```bash
  gh codespace list
  gh codespace code -R OWNER/REPO     # open in VS Code
  gh codespace ssh -R OWNER/REPO      # or use a terminal
  ```

## 3. Confirm you are inside the Codespace

```bash
echo "$CODESPACES"        # prints: true
echo "$CODESPACE_NAME"    # prints the Codespace name
pwd                       # normally /workspaces/<repository>
git branch --show-current
```

If `CODESPACES` is empty you are on another machine; do not run the setup sequence there.

## 4. Add the model key as a Codespaces secret

Never put the key in a file in the repository. Create it as a **user** Codespaces secret restricted to this repository.

```bash
gh auth refresh -h github.com -s codespace:secrets    # only if gh reports a missing scope
gh secret set GROQ_API_KEY --user --repos OWNER/REPO  # paste the value at the prompt
```

Alternatively use the repository's **Settings → Secrets and variables → Codespaces** page, or your account's Codespaces settings.

- Do not pass the value on the command line (`--body`), which can leave it in shell history.
- A running Codespace may not see a new or changed secret. Stop and start it, or rebuild it, after changing a secret.
- Use your personal Groq free-tier key for individual development. A single operator-managed key is used for the shared pilot; do not create extra keys to bypass quotas.
- Check inside the Codespace without printing the value:

  ```bash
  test -n "$GROQ_API_KEY" && echo "GROQ_API_KEY is set" || echo "GROQ_API_KEY is missing"
  ```

## 5. Create your local environment file

```bash
cp .env.example .env
```

`.env` is git-ignored and may hold local, non-secret development values. Keep real secrets in Codespaces secrets where possible.

## 6. Rebuild a Codespace

Rebuilding recreates the container from the repository's dev container definition. Files in the workspace and uncommitted changes are preserved; anything installed outside the workspace is not.

### In VS Code or the browser

Open the Command Palette and run:

- **Codespaces: Rebuild Container** — reuses cached layers. Use it after a dev container or Compose change.
- **Codespaces: Full Rebuild Container** — also discards cached images. Use it when the normal rebuild leaves a stale or broken environment.

### With the GitHub CLI

```bash
gh codespace list
gh codespace rebuild -c <codespace-name>           # normal rebuild
gh codespace rebuild -c <codespace-name> --full    # full rebuild
```

After a rebuild:

1. Check `echo "$CODESPACES"` and `git status`.
2. Check `GROQ_API_KEY` is still available (section 4).
3. Re-run the project's setup, import and reset commands (added in `W1-01`). Do not rely on database contents surviving a rebuild; the database is recreated from migrations and the import command.

Rebuild when you change the dev container, Compose files or system dependencies, or after an environment failure. Dependency changes within the workspace normally do not need a rebuild.

## 7. Work across weekly branches

Use one Codespace across the cumulative week branches. Commit before switching and follow [branching.md](./branching.md). When moving to an earlier checkpoint, use the checkpoint's reset procedure and migrations instead of editing the database by hand.

Keep forwarded ports **private**. Do not make development ports public (the plan requires that development ports stay private).

## 8. Stop, delete and recreate

Check that no work is unpushed first:

```bash
git status
git log --branches --not --remotes --oneline   # commits not pushed anywhere
```

Then:

```bash
gh codespace stop -c <codespace-name>
gh codespace delete -c <codespace-name>
```

A deleted Codespace loses everything outside the pushed branches. To recover from a corrupted environment, push your work, delete the Codespace and create a new one from the same branch (section 2).

Also use the idle timeout and retention period (`--idle-timeout`, `--retention-period` or your account's Codespaces settings) so forgotten Codespaces stop and expire.

## 9. Troubleshooting

| Symptom | Check |
|---|---|
| Creation fails or hangs | `gh codespace logs -c <name>`; confirm repository access and entitlement; retry with a new Codespace |
| Missing `GROQ_API_KEY` | Confirm the secret includes this repository, then stop/start or rebuild |
| Out of memory or slow startup | Record the measurement and consider a larger machine; this affects quota use |
| Wrong branch | `git branch --show-current`; commit or stash before switching |
| Stale environment after dev container changes | Run a full rebuild |
