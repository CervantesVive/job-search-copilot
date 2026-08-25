# job-search-copilot

A one-plugin marketplace hosting **job-search-copilot** — see
[`plugins/job-search-copilot/README.md`](plugins/job-search-copilot/README.md)
for what it does.

**Before you install:** this plugin needs [Claude
Cowork](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork),
which requires a Pro, Max, Team, or Enterprise plan — it is **not** available
on the Free plan. If you're on Free, this plugin won't work for you yet.

## Install it — two ways

**Add this repo as a marketplace (recommended — gets future updates):**

In Cowork: **Customize → Plugins → the "+" under Personal plugins → Add
marketplace → Add from a repository**, enter this repo
(`CervantesVive/job-search-copilot` or the full GitHub URL), then install
`job-search-copilot` from the list. When this repo gets updated, click
**Update** on the marketplace in Cowork to pull the new version.

**Download the plugin file directly (no repo/marketplace involved):**

Grab the `.plugin` file from this repo's [Releases](../../releases) page and
upload it: **Customize → Plugins → upload option**, select the file,
install. Simplest for a one-time install with no auto-updates.

**What's next:** once installed, follow [Setup (one
time)](plugins/job-search-copilot/README.md#setup-one-time) in the plugin
README to get your tracker up and running.

## Maintaining this repo

- The installable plugin lives in `plugins/job-search-copilot/`.
- `.claude-plugin/marketplace.json` at the repo root is what makes "Add
  marketplace" work — it points at that folder.
- After changing anything under `plugins/job-search-copilot/`, bump the
  `version` in `plugins/job-search-copilot/.claude-plugin/plugin.json`,
  commit and push, then re-zip that folder's contents (see below) and attach
  it to a new GitHub Release so the direct-download path stays current too.
- Or run the **Release plugin** workflow (Actions tab → Release plugin →
  Run workflow) to do all of the above automatically: it bumps the patch
  version, tags the commit, and publishes a GitHub Release with the zip
  attached. It fails if there are no changes under
  `plugins/job-search-copilot/` since the last release.

### Cutting a new Release zip

```bash
cd plugins/job-search-copilot
zip -r ../../job-search-copilot.plugin . -x "*.DS_Store"
cd ../..
# then attach job-search-copilot.plugin to a new GitHub Release
```
