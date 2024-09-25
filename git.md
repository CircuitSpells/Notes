# Git

## Quick Reference

start a new local repo:
`git init`

create a new branch based on master:
`git switch -c <new-branch-name>`

create a new branch without switching to it:
`git branch <branch-name>`

pull down a branch from origin without switching to it:
`git fetch origin <branch-name>`

delete local branch that has not been pushed to remote:
`git branch -D <branch-name>`
(`-D` is short for `--delete --force`)

delete local branch that has been pushed to remote:
`git branch -d <branch-name>`

view local and remote branches:
`git branch -a`

push current branch and set remote as upstream:
`git push --set-upstream origin <branch-name>`

see which local branches are tracking a remote branch:
`git branch -vv`

unstage files:
`git restore --staged <directory-or-file-path>`

discard changes to unstaged files:
`git restore <directory-or-file-path>`

reset files to match the main branch:
`git checkout <main-branch> -- <directory-or-file-path>`

view diff of a file:
`git diff <path-to-file>`
or to see diff of all files:
`git diff .`
Note: git requires the program `less` to use diff. If it is not installed, run the following, then close and reopen the terminal so that it references the updated PATH:
`winget install jftuga.less`

view upstream branch:
`git remote -v`

see recent commits:
`git log`

see a compact view of past commits:
`git log --oneline`

see the commit history for a single file:
`git log -- <file-name>`

view all commits since the feature branch branched from main:
`git log <main-branch>..<feature-branch>`

search commits that contain a commit message (useful with conventional commits), such as "feat:":
`git log --grep='^feat:' --since="1 month ago" --oneline --regexp-ignore-case`

show what changed in a specific commit:
`git show <commit-id>`

diff between main and your feature branch:
`git diff <main-branch>..<feature-branch>`
or `git diff <main-branch>..<feature-branch> -- <directory-or-file-path>`

diff between main branch and feature branch, but only compare the tip commits between both branches (note the three dots `...`):
`git diff <main-branch>...<feature-branch>`
or `git diff <main-branch>...<feature-branch> -- <directory-or-file-path>`

checkout a specific commit:
`git checkout <commit-hash>`

merge feature branch into main:
`git switch main`
`git pull`
`git merge <feature-branch>`
or, optionally: `git merge --squash <feature-branch>`
`git commit -m "merged feature branch into main"`

abort a merge in progress / permanently remove all uncommitted changes:
`git reset --hard HEAD`

undo and delete the last N commits:
`git reset --hard HEAD~<N>`
where N is the number of commits, e.g.: `git reset --hard HEAD~1`

undo the last N commits, but keep the changes from the undone commits in the staging area:
`git reset --soft HEAD~<N>`
where N is the number of commits, e.g.: `git reset --soft HEAD~1`

get current git username and email:
`git config --get user.email`
`git config --get user.name`

set local git username and email:
`git config user.email "your.email@example.com"`
`git config user.name "Your Name"`

## Advanced Operations

view the common ancestor node between two branches (for example, to see when a feature branch stemmed off of main):
- find the common ancestor commit id: `git merge-base branch1 branch2`
- see commit details (including date, commit message, etc.): `git show <commit-id>`

if you accidentally made a commit to local main (but haven't pushed), you can move that commit to a new feature branch:
- find the commit hash of your commit and copy it: `git log --oneline`
- create your feature branch off of that commit: `git branch <feature-branch-name> <commit-hash>`
- reset main back one commit (this cannot be undone): `git reset --hard HEAD~1`
- switch to the feature branch: `git switch <feature-branch-name>`
- push to origin: `git push --set-upstream origin <feature-branch-name>`

how to handle needing changes on a branch that is currently stuck in a PR, and squash commits are enabled for all PRs:
the problem: recall that squash commits create a brand new commit with a brand new hash. if feature branch B branched from feature branch A, and then A squash merges into main, then it is possible for code changes to occur that were unintentional when branch B squash merges into main. For example, if feature branch X changes lines that A touched (and merge conflicts were dealt with), and is squash merged into main after A but before B, then applying B could accidentally undo changes that X made when B is squash merged into main. This is because those initial commits from A that exist on B are still there, and reapply into main.
the solve:
- create feature branch B off of feature branch A
`git switch FeatureA`
`git switch -c FeatureB`
- when finished making changes on B, pull latest main:
`git switch main`
`git pull`
- create a new branch from main (this branch will eventually be the one to merge to main in the PR, so name it well)
`git switch -c FeatureB2`
- switch back to B:
`git switch FeatureB`
- start an interactive rebase:
`git rebase -i FeatureB2`
- this will open a text editor with a list of all the commits between `FeatureB` and `FeatureB2` (which currently matches `main`), something like this:
```
pick 1fc6c95 do something
pick 6b2481b do something else
pick dd1475d changed some things
pick c619268 more changes
```
- in the text editor, replace `pick` with `drop` for the commits you want to drop (the ones that have been included in the squash merge). it should look something like this:
```
drop 1fc6c95 do something
drop 6b2481b do something else
pick dd1475d changed some things
pick c619268 more changes
```
- save and close the editor. git will start the rebase, and will drop the commits where you replaced `pick` with `drop`.
- make a PR for `FeatureB2` into `main`.

## Conventional Commits

Conventional Commits are meant to go hand-in-hand with semantic versioning. See the docs [here](https://www.conventionalcommits.org/en/v1.0.0-beta.2/).

commit message structure:
```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

commit elements:
1. `fix:` correlates with PATCH in semantic versioning.

2. `feat:` correlates with MINOR in semantic versioning.

3. `BREAKING CHANGE:` *is placed at the beginning of the optional body*, and correlates with MAJOR in semantic versioning. This can be added to a commit of any type.

4. Others: `chore:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, and others. A scope may be provided to a commit’s type, e.g., `feat(parser): add ability to parse arrays`.
