# Git Revert Guide — FolioOne Accenture Portfolio

This document records how the last commit on `main` was reverted and pushed to GitHub on **May 26–27, 2026**.

| Item | Value |
|------|-------|
| Repository | [xon27/myportfolio2026](https://github.com/xon27/myportfolio2026.git) |
| Branch | `main` |
| Commit reverted | `59773a5` — "inital commit" |
| Revert commit | `526f1af` — Revert "inital commit" |
| Remote updated | Yes (`git push`) |

---

## When to use `git revert` vs `git reset`

| Situation | Recommended command |
|-----------|---------------------|
| Commit already pushed to GitHub (shared `main`) | `git revert HEAD` |
| Commit only exists locally, not pushed | `git reset HEAD~1` (or `--soft` / `--hard`) |
| You want to keep history and add an undo commit | `git revert` |
| You want to erase the commit from history (local only) | `git reset` |

For this project, the commit was already on `origin/main`, so **`git revert`** was used instead of `git reset` + force push.

---

## Step-by-step commands (what was run)

### 1. Inspect the repository

Check current branch, staged files, and recent history:

```powershell
git status
git log -3 --oneline
```

Optional — see what the last commit changed:

```powershell
git log -1 --stat
git diff --cached --stat
```

**What we found:** Last commit was `59773a5` ("inital commit"). There were staged changes that conflicted with a clean revert. Branch was up to date with `origin/main`.

---

### 2. Clean the working tree before reverting

Unstage and discard local changes so `git revert` can run cleanly:

```powershell
git restore --staged .
git restore .
```

---

### 3. Revert the last commit

Create a new commit that undoes the previous one (safe for pushed branches):

```powershell
git revert HEAD --no-edit
```

`--no-edit` skips opening an editor for the revert message.

**First attempt failed** with:

```
error: The following untracked working tree files would be overwritten by merge:
    assets/docs/dixoncarnacete.cv2026.pdf
    assets/docs/dixoncarnacete.pdf
Please move or remove them before you merge.
```

Git could not restore those files from history because copies already existed on disk as **untracked** files.

---

### 4. Fix the blocker and retry the revert

Remove the conflicting untracked files (Git will restore them from the revert):

```powershell
Remove-Item "assets/docs/dixoncarnacete.cv2026.pdf", "assets/docs/dixoncarnacete.pdf" -Force
git revert HEAD --no-edit
```

**Result:** New commit `526f1af` — `Revert "inital commit"`.

---

### 5. Verify locally

```powershell
git status
git log -3 --oneline
```

Expected:

- Working tree clean
- Branch ahead of `origin/main` by 1 commit (the revert)

---

### 6. Push to GitHub

```powershell
git push
```

Expected output:

```
59773a5..526f1af  main -> main
```

---

## Quick reference — copy/paste workflow

For a **pushed** last commit on `main`:

```powershell
# 1. Check state
git status
git log -3 --oneline

# 2. Clean local changes (if any)
git restore --staged .
git restore .

# 3. Revert (fix untracked file errors if Git reports them)
git revert HEAD --no-edit

# 4. Verify and push
git status
git log -3 --oneline
git push
```

---

## Troubleshooting

### "Untracked working tree files would be overwritten by merge"

**Cause:** Files on disk are not tracked by Git, but `git revert` needs to create or restore those same paths.

**Fix:** Move, delete, or temporarily rename the untracked files, then run `git revert` again:

```powershell
Remove-Item "path/to/conflicting-file" -Force
git revert HEAD --no-edit
```

On macOS/Linux, use `rm` instead of `Remove-Item`.

---

### Staged changes you do not want to lose

Before `git restore`, you can save work:

```powershell
git stash push -m "backup before revert"
# ... run revert ...
git stash pop
```

---

### Undo the revert (go back to the reverted commit’s state)

If you need to re-apply "inital commit" after reverting:

```powershell
git revert HEAD --no-edit
git push
```

That creates a second revert, which restores the original changes.

---

## Commit history after the operation

```
526f1af  Revert "inital commit"    ← current main (remote)
59773a5  inital commit             ← undone by revert (still in history)
6f0f686  update about section
```

The original commit `59773a5` remains in history; the revert commit adds the inverse changes on top. This is normal and preferred for shared branches.

---

## Files affected by the revert

The revert restored the project to the state before `59773a5`, including:

- `about.html`, `contact.html`, `index.html`, `mywork.html`, `resume.html`
- Restored: `assets/docs/dixoncarnacete.cv2026.pdf`, `assets/docs/dixoncarnacete.pdf`
- Removed: `assets/docs/mycv.pdf`

---

## Related Git commands (not used in this session)

| Command | Effect |
|---------|--------|
| `git reset --soft HEAD~1` | Remove last commit; keep changes staged |
| `git reset HEAD~1` | Remove last commit; keep changes unstaged |
| `git reset --hard HEAD~1` | Remove last commit and discard changes (destructive) |
| `git restore --staged .` | Unstage all files |
| `git restore .` | Discard unstaged changes in working tree |

Avoid `git push --force` on `main` unless you fully understand the impact on collaborators and deployment.
