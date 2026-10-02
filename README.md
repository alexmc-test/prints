# Print Library

This repository is a library of 3D models and the prints you make from them. Every model lives in its own folder under `models/` with a `project.json` record; every change arrives as a pull request that the library check reviews.

## Add a model

1. Open the [`inbox/`](inbox/) folder, choose **Add file → Upload files**, drop the model files and commit directly to the default branch. Browser uploads are limited to 25 MiB per file; larger files (up to 50 MiB) come in through the agent or the library tool.
2. The library bot moves each upload unchanged into a new model folder and opens a pull request with a minimal record and a list of missing information: ready for review when the Library check finds no errors, a draft while it finds some. If GitHub Actions may not open pull requests here, the bot pushes a branch instead and lists a compare link in one tracking issue.
3. Review the pull request in the viewer, complete the record, and merge. STL and OBJ files record no unit, so the bot records them in millimetres; edit the model's `project.json` if that is wrong.

Uploads to `inbox/` are the only direct commits. Everything else, including the drafted models, comes in as a pull request.

- Files committed into a model folder become a new-version pull request. A changed file under `original/` is kept as a new version and the original is restored by pull request, because downloads are never edited.
- If you upload with **Create a new branch for this commit**, the bot drafts the upload as one more commit on that branch.
- Uploading new bytes to the same `inbox/` path rebuilds its open pull request, which turns into a draft or back to ready as the check's result changes. Once you commit to its branch yourself, the bot never rewrites it and only leaves a comment.
- Closing the bot's pull request without merging tells the bot to leave that upload alone. To draft it again, run **Actions → Library inbox → Run workflow** with the upload's path as `redraft`.
- On the bot's pull requests GitHub shows "1 workflow awaiting approval": that is the *Library check request* fallback, which GitHub holds on pull requests a bot opens. You can ignore it; the bot already posted the Library check.
- Rejected uploads (unsupported, Git LFS pointers, too large) stay in `inbox/` and are listed in the tracking issue with the reason and a link to delete the file.

## Log a print, complete a model, set up the workshop

Open a new issue and choose **Print log**, **Model details** or **Workshop setup**. When the form is complete the library bot opens a pull request on `library/print/issue-<number>` that closes the issue when merged; when something is missing or misspelled it comments on the issue (label `needs-info`) and reads it again after each edit. Closing the issue closes the bot's pull request; closing the pull request without merging leaves the issue open for another edit.

- Only people with write access to this repository can trigger the bot; anyone else's issue gets no bot activity.
- Drag photos into the Photos box; they are imported with their location data removed. A photo the bot cannot import (for example a link to a file rather than a dropped image, which GitHub only serves to a signed-in browser in a private library) is listed in the pull request for you to add by hand.

## Open the library in the viewer

A public library opens in the hosted viewer at `https://print-library.alexcarvalho.me/<owner>/<repository>`, for example `https://print-library.alexcarvalho.me/octo/prints`. `viewer_origin` in `library.json` names that viewer, so pull requests link to the drafted model in it. The hosted viewer reads public libraries only: in a private library, remove `viewer_origin` (pull requests then link to GitHub), or set it to your own viewer's origin (an `https://` origin with no path).

## Work with an agent

Install the agent skill in Claude Code with one command (it ships with its own copy of the library tool):

```sh
claude plugin marketplace add AlexMC/3dhub && claude plugin install print-library@print-library
```

Inside this repository the agent runs this library's own `.library/tool.mjs`, adds models and versions with the same writers as the bot, checks its work, and opens pull requests from `library/agent/…` branches; it never merges or pushes to the default branch. Give it a fine-grained personal access token limited to this repository with **Contents** and **Pull requests** set to *Read and write* and no **Administration** or **Workflows** permission (`gh auth login --with-token`). Helper receipts it saves keep host paths in `*.local.json` files, which `.gitignore` keeps out of the repository.

## Finish setup

A new library opens a **Finish setup** issue with these steps on its first push. If that issue is missing (GitHub may not run workflows when a repository is created from a template), run **Actions → Library setup → Run workflow**, or follow the steps here.

<!-- finish-setup:begin -->
One setting is required:

1. **Let the library bot open pull requests.** In [Settings → Actions → General](../../settings/actions), under *Workflow permissions*, tick **Allow GitHub Actions to create and approve pull requests**. New repositories have it off and a template cannot turn it on. While it is off, the bot pushes a `library/draft/…` branch and lists a compare link for it in one tracking issue; open the pull request from that link yourself.

Good to know:

- **Mind the upload limit.** The browser uploads at most 25 MiB per file. Files between 25 and 50 MiB come in through the agent or the library tool; the check rejects files above 50 MiB and Git LFS pointers.
- **Before making the library public**, know that the viewer may load model files and thumbnails of public libraries through the jsDelivr CDN, which keeps copies permanently, even after you delete a file or make the repository private. Print logs and photos are never loaded through jsDelivr.
- **In a public library, record each downloaded model's license.** A new model merges without one, but the next change to it fails the Library check (`LICENSE_REQUIRED_TO_CHANGE`) until its license is recorded. The bot's pull request links the **Complete model details** form for it.
- **Open the library in the viewer** at `https://print-library.alexcarvalho.me/<owner>/<repository>` once it is public. `viewer_origin` in `library.json` makes pull requests link there; remove it in a private library.

### Protect your library (optional)

Without these settings every pull request still gets the Library check and shows its errors, but nothing stops a merge. To have GitHub block merging a pull request with errors:

1. **Require the library check before merging.** In [Settings → Rules → Rulesets](../../settings/rules), create a branch ruleset for the default branch that turns on **Require status checks to pass** with the check **`Library check`**, plus **Block force pushes** and **Restrict deletions**. Add **Repository admin** to the bypass list so your browser uploads to `inbox/` still commit directly; any other bypass is a visible, deliberate act.
2. **Know where errors block merge.** GitHub enforces rulesets on private repositories only on paid plans. On the Free plan the check still reports every error on each pull request, but merging a private library's pull request is not blocked; public libraries enforce the ruleset.
3. **Protect the check in a public library.** The check runs from your default branch through the `pull_request_target` trigger, which a pull request cannot switch off. GitHub blocks that trigger by default in public repositories from November 2, 2026; in [Settings → Actions → Policies](../../settings/actions), add an event policy that allows `pull_request_target` for `.github/workflows/check.yml`. Without it the check falls back to a trigger that a pull request can remove while faking a passing `Library check` of its own. In [Settings → Actions → General](../../settings/actions), under *Fork pull request workflows from outside collaborators*, choose **Require approval for all outside collaborators** and read each pull request's workflow changes before approving; leave sending write tokens or secrets to fork pull request workflows off.
<!-- finish-setup:end -->

## What is in this repository

- `models/<slug>/` — one folder per model: `project.json`, the untouched downloads under `original/`, versions and print logs.
- `inbox/` — drop uploads here.
- `library.json` — the library's name, viewer origin and links to projects kept in their own repositories.
- `workshop.json` — your printer, nozzle, materials and slicer presets, shared by every print you log.
- `.library/tool.mjs` — the library tool. Every workflow checks it against a pinned SHA-256 and refuses to run a changed copy; only the owner's `tool update` pull request may change it.
- `.github/` — the library check, the inbox automation, setup and issue forms.
