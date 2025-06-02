# Git

## Basic CLI Operations

### git init

start a new local repo:

```
git init
```

### git switch

switch to existing branch:

```
git switch <branch-name>
```

create a new branch based on the current branch:

```
git switch -c <new-branch-name>
```

switch to a branch that only exists on origin:

```
git fetch
git switch -c <local-branch-name> --track origin/<branch-name>
```

switch to a commit:

```
git switch --detach <commit-hash>
```

### git fetch

pull a branch from origin without switching to it:

```
git fetch origin <origin-branch-name>:<local-branch-name>
```

> e.g.
>
> ```
> git fetch origin myBranch:myBranch
> ```

### git add

stage files:

```
git add <directory-or-file-path>
```

stage multiple files:

```
git add <file1> <file2> <file3>
```

### git restore

unstage files:

```
git restore --staged <directory-or-file-path>
```

reset files to match the last commit (discard changes):

```
git restore <directory-or-file-path>
```

reset files to match the main branch:

```
git restore --source <main-branch> <directory-or-file-path>
```

### git status

see tracked/untracked and staged/unstaged files:

```
git status
```

### git commit

commit staged files to local branch:

```
git commit -m "commit message"
```

### git push

push to local branch:

```
git push
```

push current branch and set remote as upstream:

```
git push --set-upstream origin <branch-name>
```

### git pull

pull remote changes into your local branch:

```
git pull
```

### git branch

view local branches:

```
git branch
```

view remote branches:

```
git branch -r
```

view local and remote branches:

```
git branch -a
```

create a new branch without switching to it:

```
git branch <branch-name>
```

delete local branch that has not been pushed to remote (`-D` is short for `--delete --force`):

```
git branch -D <branch-name>
```

delete local branch that has been pushed to remote:

```
git branch -d <branch-name>
```

see which local branches are tracking a remote branch:

```
git branch -vv
```

see which branches contain a specific commit:

```
git branch --contains <commit-hash>
```

> note: add `-r` after `branch` to list remote branches or `-a` for both local and remote

### git remote

view upstream branch:

```
git remote -v
```

### git diff

> note: git requires the program `less` to use diff. If it is not installed, run the following, then close and reopen the terminal so that it references the updated PATH:
>
> ```
> winget install jftuga.less
> ```

view diff of a file:

```
git diff <path-to-file>
```

view diff of all files:

```
git diff .
```

diff between main and your feature branch:

```
git diff <main-branch>..<feature-branch>
```

diff between main and your feature branch (specify path):

```
git diff <main-branch>..<feature-branch> -- <directory-or-file-path>
```

diff between main branch and feature branch, but only compare the tip commits between both branches (note the three dots `...`):

```
git diff <main-branch>...<feature-branch>
```

same as above but also specify path:

```
git diff <main-branch>...<feature-branch> -- <directory-or-file-path>
```

### git log

see recent commits:

```
git log
```

see a compact view of recent commits:

```
git log --oneline
```

see commit graph:

```
git log --graph --decorate --oneline
```

see the commit history of a file:

```
git log -- <file-name>
```

see all commits filtered by path:

```
git log -- <directory-path>
```

view all commits since the feature branch branched from main (or between two commits on the same branch):

```
git log <main-branch>..<feature-branch>
```

```
git log <start-commit>..<end-commit>
```

search commits that contain a commit message (useful with conventional commits), such as "feat:":

```
git log --grep='^feat:' --since="1 month ago" --oneline --regexp-ignore-case
```

### git show

show what changed in a specific commit:

```
git show <commit-id>
```

### git merge

merge branchB into branchA:

```
git switch branchA
git pull
git merge branchB
```

> or, optionally:
>
> ```
> git merge --squash branchB
> ```

### git reset

abort a merge in progress and permanently remove all uncommitted changes:

```
git reset --hard HEAD
```

undo and delete the last N commits:

```
git reset --hard HEAD~<N>
```

> where N is the number of commits, e.g.:
>
> ```
> git reset --hard HEAD~1
> ```

undo the last N commits, but keep the changes from the undone commits in the staging area (use this if you accidentally commit to main):

```
git reset --soft HEAD~<N>
```

> where N is the number of commits, e.g.:
>
> ```
> git reset --soft HEAD~1
> ```

### git revert

undo a commit from origin and add that undo as a new commit:

```
git revert <commit-hash> --no-edit
git push
```

### git config

get current git username:

```
git config --get user.email
```

get current git email:

```
git config --get user.name
```

set local git username:

```
git config user.email "your.email@example.com"
```

set local git email:

```
git config user.name "Your Name"
```

set pruning to true (remove tracking to branches that have been deleted on origin):

```
git config remote.origin.prune true
```

set an alias

```
git config --global alias.<alias-name> '<git-command>'
```

> alias to create a shorthand for 'git status', e.g.:
>
> ```
> git config --global alias.gs 'status'
> git gs
> ```
>
> remove `--global` if you want the alias to be local. Edit aliases in the .gitconfig file.

view aliases

```
git config --global --get-regexp alias
```

### git cherry-pick

to cherry pick a commit, switch to the branch that the commit will be added to and then run:

```
git cherry-pick <commit-hash>
```

cherry pick a range of commits:

```
git cherry-pick <start-commit-hash>^..<end-commit-hash>
```

> The ^ symbol after the start commit indicates that you want to include the start commit in the range. In the above example, both the start and end commits will be included

if the cherry-pick has merge conflicts, you can resolve them in a text editor or abort the changes:

```
git cherry-pick --abort
```

### git stash

stash tracked/untracked, staged/unstaged files. All files will be stashed as unstaged (message is optional):

```
git stash -u -m "my message"
```

stash tracked/untracked but don't stash staged (message is optional):

```
git stash push -u --keep-index -m "my message"
```

list stashes:

```
git stash list
```

view stash contents (omit stash name to view most recent stash `stash@{0}`):

```
git stash show -p "stash@{2}"
```

apply a specific stash by index (omit stash name to apply most recent stash `stash@{0}`):

```
git stash apply stash@{2}
```

delete stash (omit stash name to delete most recent stash `stash@{0}`):

```
git stash drop "stash@{2}"
```

delete all stashes:

```
git stash clear
```

retrieve dropped stash: https://stackoverflow.com/questions/65182172/visual-studio-undo-drop-stash

### git clean

print out list of files and directories which will be removed without removing them:

```
git clean -d --dry-run
```

delete files that are not under version control:

```
git clean -fd
```

### git update-index

hide local edits to _tracked_ files (essentially your own personal .gitignore):

```
git update-index --assume-unchanged <directory-or-file-path>
```

> note: to see which files are no longer being tracked due to --assume-unchanged, run the following:
>
> ```
> git ls-files -v | Select-String '^h' -CaseSensitive
> ```

to undo the `--assume-unchanged` command:

```
git update-index --no-assume-unchanged <directory-or-file-path>
```

> note: to ignore _untracked_ files, add them to the `.git/info/exclude` file.

## Advanced CLI Operations

### View Common Ancestor Commit

view the common ancestor commit between two branches (i.e. when a feature branch forked from main):

find the common ancestor commit id:

```
git merge-base branch1 branch2
```

see commit details (including date, commit message, etc.):

```
git show <commit-id>
```

### Move Commits On Main To A New Branch

if you accidentally made commits to main (but haven't pushed), then you can move those commits to a new feature branch (if you have only added one commit to main, consider using `git reset --soft HEAD~1` instead):

find the commit hash of oldest accidental commit to main and copy it:

```
git log --oneline
```

create your feature branch off of that commit:

```
git branch <feature-branch-name> <commit-hash>
```

switch to the feature branch:

```
git switch <feature-branch-name>
```

For each commit after the oldest accidental commit, run git cherry-pick in order of the oldest commit to the newest (there is also a way to add commit ranges but this is less error-prone):

```
git cherry-pick <commit-hash>
```

switch back to main:

```
git switch main
```

reset main back N commits where N is the number of commits accidentally added to main (this cannot be undone):

```
git reset --hard HEAD~<N>
```

switch to the feature branch:

```
git switch <feature-branch-name>
```

### Continue Work on Pending Changes in PR

when you start a new branch, sometimes you depend on changes that are currently on another branch that is stuck in a PR. You can use the initial branch's commits to develop off of and then cherry pick the new commits afterward to keep things clean:

If branch FeatureA is stuck in a PR, create branch FeatureB off of branch FeatureA:

```
git switch FeatureA
git pull
git switch -c FeatureB
```

the following assumes that the PR for FeatureA has completed at this point. when finished making changes on branch FeatureB or whenever you'd like to resync main into FeatureB, pull latest:

```
git switch main
git pull
```

create a new branch from main (this branch will eventually be the one to merge to main in the PR, so name it accordingly)

```
git switch -c FeatureB2
```

switch back to FeatureB:

```
git switch FeatureB
```

start an interactive rebase to add the new commits from FeatureB onto FeatureB2:

```
git rebase -i FeatureB2
```

this will open a text editor with a list of all the commits between `FeatureB` and `FeatureB2` (which currently shares a HEAD with `main`), something like this:

```
pick 1fc6c95 do something
pick 6b2481b do something else
pick dd1475d changed some things
pick c619268 more changes
```

in the text editor, replace `pick` with `drop` for the commits you want to drop (the ones that have been included in the PR from FeatureA). it should look something like this:

```
drop 1fc6c95 do something
drop 6b2481b do something else
pick dd1475d changed some things
pick c619268 more changes
```

save and close the editor. git will start the rebase, and will drop the commits where you replaced `pick` with `drop`. then, make a PR for `FeatureB2` into `main`.

## Conventional Commits

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) are meant to go hand-in-hand with [semantic versioning](https://semver.org/).

commit message structure:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

commit elements:

1. `fix:` correlates with PATCH in semantic versioning.

2. `feat:` correlates with MINOR in semantic versioning.

3. `BREAKING CHANGE:` _is placed at the beginning of the optional body_, and correlates with MAJOR in semantic versioning. This can be added to a commit of any type.

4. Others: `chore:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, and others. A scope may be provided to a commit’s type, e.g., `feat(parser): add ability to parse arrays`.
