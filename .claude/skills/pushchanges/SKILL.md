---
name: pushchanges
description: "Stage changes, commit, push branch, and create a PR against main"
allowed-tools: Bash(git *), Bash(gh *)
argument-hint: "[optional PR title/description hint]"
---

## Context
- Current branch: !`git branch --show-current`
- Remotes: !`git remote -v`
- Status: !`git status --short`

## Task
User notes (may be empty): $ARGUMENTS

Perform the following git and PR automation steps cleanly:

0. **Pre-checks** — stop with a clear message if any of these fail:
   - There are no changes to ship (status above is empty).
   - There is no `origin` remote.
   - `gh auth status` reports you are not logged in.

1. **Check Branch**:
   - If currently on `main` or `master`, automatically create and switch to a new branch named `<type>/<short-description>` based on the changes (e.g. `feat/add-login`, `fix/null-check`).
   - Otherwise, stay on the current branch.

2. **Stage Changes**:
   - Review `git status` and stage files explicitly by path (`git add <file> ...`).
   - Never stage `.env*`, credentials/keys, `node_modules/`, build output (`dist/`, `build/`), or OS files (`.DS_Store`).
   - If a file looks sensitive or you are unsure whether it belongs, stop and ask the user.

3. **Commit Changes**:
   - Inspect `git diff --cached`.
   - Draft a clear, concise commit message following conventional commits format (`feat:`, `fix:`, `chore:`, ...) based on the staged diff.
   - Incorporate the user notes above into the commit/PR context if provided.
   - Commit the changes using a heredoc so multi-line messages are preserved:
     ```
     git commit -F - <<'EOF'
     <message>
     EOF
     ```

4. **Push Branch**:
   - Push the current branch and set upstream: `git push -u origin <current-branch>`.
   - Never force-push.

5. **Create Pull Request**:
   - Create a Pull Request against `main` using the GitHub CLI, passing the body via heredoc:
     ```
     gh pr create --base main --title "<Title>" --body-file - <<'EOF'
     ## Summary
     <what changed and why>

     ## Changes
     - <bullet list of key changes>
     EOF
     ```
   - Output the resulting Pull Request URL clearly to the user.
