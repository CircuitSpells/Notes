# Git

## Terminology

- `HEAD`: a reference to the current commit that you are pointing to.
- `Index`: staging area; what will be committed next.
- `Working Tree`: your actual files on disk.

## Setup

to set up GitHub auth on a Linux machine:

generate an ssh token:

```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_<some-name-you-choose> -C "<email>"
```

> e.g. `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_personal -C "personal@example.com"`

you will be prompted for an optional passphrase. This will be required for every shell session if you choose to make one.

start ssh agent and add token:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_<the-name-you-chose>
```

copy the contents of `~/.ssh/id_ed25519_<the-name-you-chose>.pub` (note the ".pub") to your clipboard.

log into your GitHub account:

- click your profile picture in the top right.
- select settings.
- select "SSH and GPG keys".
- select "New SSH key".
- give a descriptive title, set Key type to "Authentication Key", and paste the key contents into the Key field.
- select "Add SSH key".
- complete verification steps.

create a file called `config` in the `~.ssh/` directory and add a Host like so:

```
Host <a-hostname-you-choose>
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_<the-name-you-chose>
    IdentitiesOnly yes
```

add the identity to known-hosts:

```
ssh -T git@<the-hostname-you-chose>
```

when prompted to continue connecting, type `yes` and enter. You should get a message saying you've successfully authenticated, but GitHub does not provide shell access.

go to your repo directory and update the origin url:

```
git remote set-url origin git@<the-hostname-you-chose>:<github-username>/<repo-name>.git
```

verify origin was updated:

```
git remote -vv
```

make sure to set your email and name in the local git config:

```
git config user.email "<email>"
git config user.name "<name>"
```

optionally set default pull, rebase, and push behavior:

```
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global push.autoSetupRemote true
```

you can now do git operations on the remote branch! Repeat this process for any additional GitHub accounts you need access to.

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
git switch -c <branch-name>
```

switch to a branch that only exists on origin:

```
git fetch
git switch -c <local-branch-name> --track origin/<branch-name>
```

create a new branch directly from upstream main (as opposed to local main):

```
git switch -c <branch-name> upstream/main
```

switch to a commit:

```
git switch --detach <commit-hash>
```

### git fetch

pull a branch from origin without switching to it (note that this will not set the upstream branch--this needs to be manually):

```
git fetch origin <origin-branch-name>:<local-branch-name>
```

> e.g.
>
> ```
> git fetch origin myBranch:myBranch
> ```

### git checkout

pull a branch from origin, switch to it, and automatically set its upstream branch:

```
git checkout --track origin/<branch-name>
```

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

see tracked/untracked and staged/unstaged files, and see upstream branch:

```
git status
```

> note: if there is an upstream branch, the message `Your branch is up to date with 'origin/<upstream-branch-name>'` will appear, otherwise no message will appear.

### git commit

commit staged files to local branch:

```
git commit -m "commit message"
```

amend last commit (apply staged changes to most recent commit):

```
git commit --amend
```

edit last commit message:

```
git commit --ammend -m "new commit message"
```

> note: only use --ammend if you have not yet pushed the latest commit, otherwise --force-with-lease will be required.

> note: in order to edit commits older than the most recent, see details for `git rebase --interactive` wit the 'edit' option.

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

pull changes from a remote branch (such as an upstream fork):

```
git pull <remote-name> <branch-name>
# e.g.
# git pull upstream main
```

rebase local commits on top of pull (instead of creating a merge commit, allows for cleaner git history):

```
git pull --rebase
```

auto stash local changes, then reapply them after the rebase (merge conflicts will need to be resolved manually):

```
git pull --rebase --autostash
```

set the above as the default behavior globally:

```
git config --global pull.rebase true
git config --global rebase.autoStash true
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

view remote origin and upstream branch:

```
git remote -vv
```

set origin repo (should be set automatically when you clone):

```
git remote set-url origin https://github.com/<github-username>/<repo-name>.git
```

set upstream repo (to pull updates from a fork):

```
git remote add upstream https://github.com/<github-username>/<repo-name>.git
```

### git diff

> note: git requires the program `less` to use diff.

view diff of working tree relative to index (staging area):

```
git diff
```

view diff of index relative to HEAD:

```
git diff --staged
```

view diff of working tree + index (i.e. all uncommitted changes) relative to HEAD:

```
git diff HEAD
```

view diff of current commit relative to the past N commits on the current branch:

```
git diff HEAD~<N>
```

view diff of working tree + index (i.e. all uncommitted changes) relative to main:

```
git diff main
```

view diff of a file:

```
git diff <path-to-file>
```

view diff files in the current directory:

```
git diff .
```

diff between the tip of main and HEAD of a feature branch:

```
git diff <main-branch>..<feature-branch>
```

or if you're already on the feature branch, simply:

```
git diff main
```

diff between main and your feature branch (specify path):

```
git diff <main-branch>..<feature-branch> -- <directory-or-file-path>
```

diff between the tip of your feature branch and the base commit on main where the feature branch had branched from (note the three dots `...`):

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
git merge branchB --no-edit
```

`--no-edit` uses the automatic commit message, otherwise you can use `-m "my message..."`

> or, optionally:
>
> ```
> git merge --squash branchB
> ```

### git rebase

rebase a feature branch onto main:

```
git switch main && git pull

git switch <feature-branch>
git rebase main
# or, alternatively:
# git rebase main <feature-branch>

# then move main's pointer forward to match the feature commit:
git switch main
git merge <feature-branch>
```

if there are conflicts, you can abort with:

```
git rebase --abort
```

#### Interactive Rebase

interactive rebase for the past N commits:

```
git rebase --interactive HEAD~<N>
```

or, rebase all new commits on feature branch:

```
git rebase -i main
```

> note: before doing a complicated interactive rebase, consider backing up your branch first with `git branch <branch-name>-bak`.

this will open an interactive session to replay the previous N commits, allowing you to make modifications by replacing "pick" with other keywords:

- `pick`: the default option, keeps the commit unchanged.
- `reword`: edit the commit message.
- `edit`: pause at this commit so you can change content (update files, split into multiple commits, etc.).
  - to make file changes: change the desired files, run `git add .`, then `git rebase --continue`.
  - to split into multiple commits: run `git reset HEAD~1`, then add and commit changes separately, then run `git rebase --continue`.
- `squash`: combine this commit with the previous one (the commit above it). You'll get a chance to edit the commit message.
- `fixup`: like `squash`, but discard this commit's message and take the previous commit's message.
- `drop`: remove the commit.

you can also add `exec` inbetween commits:

- `exec`: run a shell command, e.g. `exec npm test`.

upon saving and closing the file, the interactive rebase will begin. Remember you can always abort at any time.

many of the above commands will continue automatically, but you need you can continue manually:

```
git rebase --continue
```

to collapse all commits on a feature branch into a single commit:

```
git switch <feature-branch>
git rebase -i main
```

then:

- leave oldest commit as "pick"
- change all others to "squash"
- save and close the file
- you will be prompted to edit the commit message. Optionally do so and then save and close the file.

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

retrieve dropped stash: <https://stackoverflow.com/questions/65182172/visual-studio-undo-drop-stash>

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

## Helpful Extras

- never rebase commits that have already been pushed.
- view common ancestor commit: `git merge-base branch1 branch2` (note that this might not work if certain combinations of merging, rebasing, and cherry-picking occurred).
- never use `git push --force` as it can rewrite history. However, if it is on your own personal feature branch then it is typically okay. A safer option is `git push --force-with-lease` which will fail if it will change someone else's commits.
- to add a local repo to GitHub (requires the gh CLI and first running `gh auth login`):

```
gh repo create --private --source=. --remote=origin
git push -u --all
gh browse
```

### Cherry Picking Strategy

generally, directly cherry-picking commits from remote branches should be avoided, and a merge should be done instead. This is because cherry-picking creates a new commit hash and git is unable to tell that the two different hashes are related. This can lead to unintended behavior with no merge conflicts. Try to only cherry-pick commits from orphaned or stale branches. If you need a commit that someone else pushed to remote:

- have the author create a separate branch from the base commit that their feature branch stems from: `git checkout <base-hash> && git checkout -b patch-branch`.
- the author then cherry-picks the commit you need to the new branch: `git cherry-pick <hash>`.
- the author pushes the branch to remote: `git add -A && commit -m "my message" && git push`.
- the author merges the patch branch back to their feature branch (this will create an empty commit): `git switch feature-branch && git merge patch-branch`.
- you merge in the patch branch: `git fetch origin && git merge origin/patch-branch`.

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
