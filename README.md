# fork-sync

Keeps my GitHub forks up to date with their upstream repos.

[`sync-forks.yml`](.github/workflows/sync-forks.yml) runs every hour:

1. **Find**: lists every fork on the account and checks each one against its upstream with the GitHub API. Nothing is cloned in this step.
2. **Sync**: for each fork that's behind, checks it out and runs [`aormsby/Fork-Sync-With-Upstream-action@v3.4.3`](https://github.com/aormsby/Fork-Sync-With-Upstream-action), which pulls from upstream and pushes to the fork's default branch.

Before pushing, the sync job turns off GitHub Actions on the fork. Otherwise each sync would run upstream's CI in the fork, which fails without upstream's secrets and uses up Actions slots. Excluded forks are never touched.

New forks are picked up automatically. Each run's summary lists which forks were synced and which were skipped.

## Excluding a fork

Add the repo name to `EXCLUDE` at the top of the workflow (names separated by spaces):

```yaml
env:
  EXCLUDE: ogre-battle-64-recomp-windows claw-code
```

`claw-code` is excluded because GitHub has disabled git access to it ("Repository is disabled").

## What gets skipped

- Repos listed in `EXCLUDE`
- Archived forks
- Forks whose upstream has been deleted or made private
- Forks with no shared history with upstream (upstream rewrote its history). These need a manual reset.

## Token

The workflow needs a repo secret named `FORK_SYNC_TOKEN`: a classic personal access token with the `repo` and `workflow` scopes. The built-in `GITHUB_TOKEN` can't push to other repos. The `workflow` scope is required because many upstream commits change `.github/workflows` files.
