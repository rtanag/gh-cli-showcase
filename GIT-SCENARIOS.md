# Git and GitHub CLI: Everyday Use and Critical Scenarios

**This is a reference and study plan.** None of these commands count as "done" until you run them in the project. Only commands you actually ran move into `DOCUMENTATION.md`.

Key terms: `HEAD` is your current commit. `origin` is the GitHub remote. `<sha>` is a commit hash from `git log --oneline`.

---

## 1. Everyday commands

| Command | Usage | When Used | Why It's Used |
|---|---|---|---|
| `git diff` | Shows unstaged changes | Before staging | Review exactly what you changed |
| `git diff --staged` | Shows staged changes | Before committing | Last check of what the commit will contain |
| `git add <file>` | Stages one file | Building a focused commit | Commits stay small and single-purpose |
| `git restore --staged <file>` | Unstages a file, keeps edits (modern form of `git reset <file>`) | Staged the wrong file | Fixes staging without losing work |
| `git restore <file>` | Discards unstaged edits to a file | You want the last committed version back | Undo local edits. **Destructive: edits are gone** |
| `git rm <file>` | Deletes a file and stages the deletion | Removing a tracked file | Records the removal in one step |
| `git rm --cached <file>` | Stops tracking a file but keeps it on disk | A file should be ignored (for example `.env`) | Removes it from Git without deleting it locally |
| `git mv <old> <new>` | Renames or moves a file and stages it | Renaming a tracked file | Git keeps the rename visible in history |
| `git log --oneline --graph --all` | Compact history graph of all branches | Understanding how branches relate | Quick visual of the branch structure |
| `git log --stat -M` | Log with changed files, detecting renames | Seeing what each commit touched | Shows file-level impact and moved paths |
| `git log --follow <file>` | History of one file across renames | Tracing a file's past | Does not lose history after a rename |
| `git show <sha>` | Shows one commit's message and changes | Inspecting a specific commit | Understand what a commit did |
| `git blame <file>` | Shows who last changed each line | Finding when and why a line changed | Traces a bug to its commit and author |
| `git config --global color.ui auto` | Colors Git output | One-time setup | Easier to read diffs and status |
| `git config --global core.excludesfile <file>` | A global ignore file for all repos | Ignoring OS or editor files everywhere | Keeps personal junk out of every project's `.gitignore` |

---

## 2. Undoing things (pick by situation)

| Situation | Command | Safe on pushed commits? | Why It's Used |
|---|---|---|---|
| Edited a file, want it back | `git restore <file>` | n/a (local only) | Discards working-tree edits |
| Staged the wrong file | `git restore --staged <file>` | n/a | Unstages, keeps edits |
| Last commit message or content is wrong, not pushed | `git commit --amend` | **No** | Rewrites the last commit |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` | **No** | Re-do the commit differently |
| Undo last commit, keep changes unstaged | `git reset --mixed HEAD~1` (default) | **No** | Re-stage selectively |
| Undo last commit and throw changes away | `git reset --hard HEAD~1` | **No** | Full local rollback. **Destructive** |
| Undo a commit that is already pushed | `git revert <sha>` | **Yes** | Adds a new commit that cancels the old one, history stays intact |
| Undo a merge commit that is already pushed | `git revert -m 1 <sha>` | **Yes** | `-m 1` keeps the main-line side of the merge |
| Remove untracked files | `git clean -n` (preview) then `git clean -fd` | n/a | Cleans the working folder. **Destructive: always preview first** |

**Rule of thumb:** local and unpushed means you can rewrite (`amend`, `reset`). Pushed and shared means you add (`revert`).

---

## 3. Aborting and escaping a mess

| Situation | Command | Why It's Used |
|---|---|---|
| Merge produced conflicts and you want out | `git merge --abort` | Returns to the state before the merge started |
| Rebase went wrong | `git rebase --abort` | Returns to the state before the rebase |
| Rebase conflict resolved, keep going | `git add <file>` then `git rebase --continue` | Continues after fixing a conflict |
| Skip a commit during a rebase | `git rebase --skip` | Drops the problematic commit |
| Cherry-pick conflicted | `git cherry-pick --abort` | Cancels the cherry-pick |
| Revert conflicted | `git revert --abort` | Cancels the revert |
| Bisect is finished | `git bisect reset` | Returns to the branch you started on |
| Not sure what state you are in | `git status` | Always says what is in progress and how to abort or continue |

---

## 4. Merge conflicts

| Step | Command | Why It's Used |
|---|---|---|
| See conflicted files | `git status` | Lists files under "Unmerged paths" |
| Open the file and find the markers | `<<<<<<<`, `=======`, `>>>>>>>` | The two versions sit between the markers. Edit to the final result and delete the markers |
| Take one side entirely | `git checkout --ours <file>` or `git checkout --theirs <file>` | Quick resolution when one side should win (during a merge, "ours" is your current branch) |
| Mark as resolved | `git add <file>` | Tells Git the conflict is fixed |
| Finish the merge | `git commit` (the default message is fine) | Completes the merge commit |
| Give up | `git merge --abort` | Safe exit at any time before the commit |

---

## 5. Branches not in sync (the "my change is not on the other branch" problem)

Changes on one branch never appear on another by themselves. You have to bring them over.

| Situation | Command | Why It's Used |
|---|---|---|
| See what is on GitHub without changing anything | `git fetch origin` | Downloads remote changes into `origin/*` but does not touch your branches |
| Am I ahead or behind? | `git status -sb` or `git branch -vv` | Shows `[ahead 1, behind 2]` against the upstream |
| What did I miss? | `git log HEAD..origin/main --oneline` | Commits on the remote that you lack |
| What have I not pushed? | `git log origin/main..HEAD --oneline` | Commits you have that the remote lacks |
| Update my branch (merge style) | `git pull` | Fetch plus merge |
| Update my branch (linear history) | `git pull --rebase` | Fetch plus rebase, no merge commit |
| Bring `main` into my feature branch | `git switch feature` then `git merge main` | Keeps the feature branch current |
| Same, with a linear history | `git switch feature` then `git rebase main` | Replays your commits on top of the latest `main`. Use only on unshared branches |
| Take just one commit from another branch | `git cherry-pick <sha>` | Copies a single commit instead of a whole branch |
| Branch has no upstream yet | `git push -u origin <branch>` | Pushes and sets tracking so later `git push` and `git pull` just work |
| Push rejected (remote has newer commits) | `git pull --rebase` then `git push` | Integrate first, then push. Do not force |
| Switching branches with uncommitted work | `git stash` then `git switch other` then later `git stash pop` | Parks the work so the switch is clean |
| Compare two branches | `git log B..A`, `git diff B...A` | Shows what A has that B does not |

---

## 6. Stash (temporary commits)

| Command | Why It's Used |
|---|---|
| `git stash` | Parks tracked changes and cleans the working tree |
| `git stash push -u -m "message"` | Also stashes untracked files and labels the stash |
| `git stash list` | Shows the stash stack |
| `git stash show -p stash@{0}` | Shows what a stash contains |
| `git stash pop` | Re-applies the newest stash and removes it |
| `git stash apply` | Re-applies but keeps the stash (useful for several branches) |
| `git stash drop` | Deletes the newest stash |

---

## 7. Recovery: "I lost my commits"

| Situation | Command | Why It's Used |
|---|---|---|
| Find lost commits (after a bad reset, rebase, deleted branch) | `git reflog` | Records every position `HEAD` has been in, for roughly 90 days by default |
| Go back to where you were | `git reset --hard HEAD@{n}` | Moves the branch back to the reflog entry |
| Rescue a commit onto a new branch (safest) | `git branch rescue <sha>` | Makes the lost commit reachable again without moving anything |
| Detached HEAD, and you made commits | `git switch -c rescue` | Gives those commits a branch before they are lost |
| Detached HEAD, no commits worth keeping | `git switch main` | Just go back |

---

## 8. "What would you do if...?"

| Scenario | What to do |
|---|---|
| **Committed to the wrong branch (not pushed)** | `git branch correct-branch` (or `git switch -c correct-branch`) to keep the commit, then on the wrong branch `git reset --hard origin/main`. Or cherry-pick the commit onto the right branch and reset the wrong one |
| **Committed a secret or `.env`, not pushed** | `git rm --cached .env`, add `.env` to `.gitignore`, then `git commit --amend` |
| **Pushed a secret** | Treat it as compromised: **rotate or revoke the secret first**. Then remove it from history (`git filter-repo` or BFG) and force-push, and tell the team. Rewriting history alone is not enough, because the secret may already be copied or scraped |
| **Pushed a bad commit to a shared branch** | `git revert <sha>` then `git push`. No force push |
| **Pushed a bad commit to your own feature branch** | Fix locally, then `git push --force-with-lease` (refuses if someone else pushed in the meantime, unlike `--force`) |
| **Wrong author on the last commit** | `git commit --amend --reset-author` |
| **Wrong author on several commits** | `git rebase -i` with `exec git commit --amend --reset-author --no-edit`, only on unpushed commits |
| **Wrong global identity** | `git config --global user.email "right@address"` (or `--local` for one repo) |
| **Fork is behind upstream** | `gh repo sync <owner>/<fork>`, or `git remote add upstream <url>`, `git fetch upstream`, `git merge upstream/main` |
| **Lost commits after a hard reset** | `git reflog` then `git branch rescue <sha>` |
| **Detached HEAD** | `git switch -c new-branch` to keep the work, or `git switch main` to leave |
| **Merge conflict in a PR** | `gh pr checkout <n>`, `git merge main`, fix the conflicts, `git add`, `git commit`, `git push` |
| **A bug appeared, but you do not know which commit caused it** | `git bisect start`, `git bisect bad`, `git bisect good <sha>`, repeat, then `git bisect reset` |
| **Pull request already merged but it broke things** | `git revert -m 1 <merge-sha>` through a new PR |
| **Branch protection blocks your push to `main`** | Push a branch and open a PR. This is the intended workflow |
| **Accidentally deleted a branch** | `git reflog` then `git branch <name> <sha>` |

---

## 9. Force pushing: when and how

| Command | Safe? | Why It's Used |
|---|---|---|
| `git push --force` | **Dangerous.** Overwrites the remote no matter what | Almost never use it |
| `git push --force-with-lease` | Safer. Fails if the remote has commits you have not seen | After rebasing or amending your own branch |
| Never force-push `main` or a shared branch | n/a | It rewrites history other people depend on |

---

## 10. Quick decision guide

| Question | Answer |
|---|---|
| Is the commit pushed? | **No:** `amend` or `reset` is fine. **Yes:** use `revert` |
| Is the branch shared? | **No:** `rebase` and `force-with-lease` are fine. **Yes:** `merge` and `revert` |
| Am I about to run something destructive (`reset --hard`, `clean -fd`, `restore`)? | Run `git status` first, and use the dry run (`git clean -n`) |
| Did I break something locally? | `git reflog` almost always has the way back |
| Not sure what is going on? | `git status`, `git log --oneline --graph --all`, `git reflog` |


## 11. Eme eme lang na new part here