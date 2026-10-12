# Contributing

How our group works together in this repo. Everyone reads this before their first commit.

## Who owns what

| Folder / file | Owner | Everyone else |
|---------------|-------|---------------|
| `server/` (including `openapi.yaml`) | Backend | Suggests changes through a pull request |
| `client/` | Frontend | Suggests changes through a pull request |
| `docs/`, `README.md` | Documentation | Writes first drafts about their own work. Documentation reviews and edits them |

"Owner" means you make sure it's correct and up to date. Other members can still edit it.

## Branches

- `main` must always work. **Nobody pushes directly to `main`.**
- Make a new branch for each task:

| Kind of work | Branch name | Example |
|--------------|-------------|---------|
| New feature | `feat/<short-description>` | `feat/list-announcements-endpoint` |
| Bug fix | `fix/<short-description>` | `fix/navbar-overlap-on-mobile` |
| Documentation | `docs/<short-description>` | `docs/add-erd` |

```bash
git switch main
git pull
git switch -c feat/list-announcements-endpoint
# ...work and commit...
git push -u origin feat/list-announcements-endpoint
```

Then open a **Pull Request** on GitHub.

> **Repo owner, one-time setup:** GitHub → Settings → Branches → add a rule for `main` → turn on "Require a pull request before merging". This stops anyone from pushing to `main` by accident.

## Commit messages

Format: `type(scope): what you did`

- **type:** `feat` (new feature), `fix` (bug fix), `docs` (documentation), `refactor` (cleanup, same behavior), `style` (formatting only), `chore` (setup, dependencies)
- **scope:** `server`, `client`, or `docs`

```
feat(server): add GET /announcements endpoint
fix(client): show error message when the API is down
docs: add ER diagram to data-model.md
chore(server): add swagger-ui-express
```

Make small commits often instead of one giant commit at the end.

## Pull requests

1. Keep each PR small: one feature or one fix.
2. Fill out the PR template. It includes a documentation checkbox, so don't skip it.
3. At least **one teammate reviews and approves** before merging.
4. After approval, the author merges and deletes the branch.

**Reviewing when it's not your part:** you don't need to be an expert. Check that it runs, that it matches `server/openapi.yaml`, and that the docs were updated.

## Never commit

- `.env` files (they hold passwords and keys). Commit `.env.example` instead.
- `node_modules/`, `.venv/`, `bin/`, `obj/`. The `.gitignore` already skips these.
- Large files (videos, zip files, big datasets).

**If you accidentally commit a password or key:** tell your group and change that password or key right away. Deleting the file in a later commit does not remove it from Git history.

## Staying in sync

Bring the latest `main` into your branch often, at least once a day while you're working:

```bash
git switch main
git pull
git switch your-branch
git merge main
```

**Merge conflict?** Open the file and look for the `<<<<<<<`, `=======`, and `>>>>>>>` markers. Keep the correct code, delete the markers, then commit. If you're not sure which version is correct, ask the owner of that file.
