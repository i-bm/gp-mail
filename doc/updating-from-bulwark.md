# Updating GP Mail from Bulwark

This checkout has two remotes. Fetch Bulwark from `origin`. Push GP Mail to `gp`.

| Remote | URL | Use |
| --- | --- | --- |
| `origin` | `https://github.com/bulwarkmail/webmail.git` | Upstream. `git fetch` and merge `origin/main`. |
| `gp` | `https://github.com/i-bm/gp-mail.git` | This fork. `git push` here. |

`gp-mail-lite` tracks `gp/gp-mail-lite`. The GitHub account `i-bm` cannot write to `bulwarkmail/webmail`, so `git push origin` returns 403. Upstream `main` is also checked out at `/Users/isaac/Projects/bulwart`.

## What this fork changes

`scripts/lite/lib.mjs` (`buildLiteConfig`) writes these defaults into `out/config.json`, which postbuild packs into `bulwark-lite-stalwart.zip`:

| Key | Default |
| --- | --- |
| `appName` | `GP Mail` |
| `jmapServerUrl` | empty (the Stalwart server that serves the page) |
| `loginCompanyName` | `Consolidated Security Data Limited` |
| `accountManagementUrl` | `/account/` |

Override them at build time with `LITE_APP_NAME`, `LITE_JMAP_SERVER_URL`, `LITE_LOGIN_COMPANY_NAME`, and `LITE_ACCOUNT_MANAGEMENT_URL`. The login page and Settings → Account already read the company name and the account link.

The edits live in `scripts/lite/lib.mjs` and `scripts/lite/build.mjs`.

## Pull Bulwark updates

On a branch that contains only those script edits:

```bash
git fetch origin
git merge origin/main
git push gp gp-mail-lite
```

Resolve conflicts in the two script files, then push to `gp`. Leave `origin` as fetch-only.

## Build the Stalwart zip in a throwaway worktree

`npm run build:lite -- --in-place` deletes server-only routes and tests from the working tree (`scripts/lite/prepare.mjs`). Those deletions are build debris. Keep them out of `gp-mail-lite`.

Build from a detached worktree of the branch that has the config change:

```bash
git fetch origin
git worktree add --detach /tmp/gp-mail-lite-build gp-mail-lite
cd /tmp/gp-mail-lite-build
npm ci
npm run build:lite -- --in-place --target=stalwart
```

The archive is `bulwark-lite-stalwart.zip` in that worktree. Check the packed config with:

```bash
unzip -p bulwark-lite-stalwart.zip config.json
```

Remove the worktree when the zip is copied out. The source checkout stays intact:

```bash
cd /Users/isaac/Projects/gp-mail-lite
git worktree remove --force /tmp/gp-mail-lite-build
```

## The current `customization` commit

Commit `50a753c8f` (`customization`) contains the two script edits and the prepare deletions (about 2,800 files). Merging `origin/main` into that commit fights every upstream change to a deleted file.

Replace that history with upstream plus only the script edits, then push the rewritten branch:

```bash
git fetch origin
git checkout gp-mail-lite
git reset --hard origin/main
```

Re-apply the `appName`, `loginCompanyName`, and `accountManagementUrl` defaults in `scripts/lite/lib.mjs`, and the matching env notes in `scripts/lite/build.mjs`. Then:

```bash
git add scripts/lite/lib.mjs scripts/lite/build.mjs
git commit -m "Write GP Mail branding into the Lite config."
git push --force-with-lease gp gp-mail-lite
```

`--force-with-lease` replaces `gp/gp-mail-lite` on GitHub. It refuses the push if someone else has added commits there since the last fetch.
