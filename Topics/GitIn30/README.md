# Christian Gunderman's Blog - Git in 30 Seconds

## What is the purpose of this document

This document is intended as a very quick and dirty, 0 to 60mph, end-to-end refresher course on Git for anyone that has used it before, but is having trouble understanding or remembering one of the particular specifics. It is organized in order from basic to advanced (CRAWL -> WALK -> RUN).

## *CRAWL* - What is Git

Git is a version control system, designed for tracking iterative changes to a tree of files. This fine grained tracking of changes is critical in the software world where there can be thousands of individuals all collaborating on the same set of files because it facilitates quality control and simultaneous development without the need for a single central 'source of truth'. In other words, it's like Google Docs, but without the need to be connected to a single tracking server. This decentralization is possible through a concept called merging, covered later in this document, that allows Git to intelligently combine changes from various users after the fact to create a single combined work.

## *CRAWL* - Anatomy of a repo

A git repo is essentially just a folder on your computer's drive with some magical properties that are consumed by your Git client (e.g.: [Git for Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases)) and stored in a hidden *.git* folder in the root of the repo. Everything inside of *.git* is essential for tracking changes to the repository and should not be modified directly.

Everything outside of the *.git* folder is your working directory, which represents *your repository at a particular point in time* plus any uncommitted changes and unstaged files. In otherwords, it is your repository as it exists at a particular commit plus any changes that you have made since your last commit and any files that are not tracked in your repository. As you work, you modify the working directory, and commit the desired changes as needed. Later, we'll discuss how to commit files, check out a particular commit, etc.

## *CRAWL* - Single branch workflow

### Tracked vs. Not Tracked

Tracked files are files that Git is actively maintaining history for. By default, all files in your repository are untracked until you have explicitly used `git add [filename]` to start tracking the file, or `git add .` to start tracking all files. You can see untracked files, as well as tracked files that have been modified by running `git status`. If a file is not tracked, its history is not recorded in Git, and the file is lost if the repository is cloned on another machine.

#### .gitignore

*.gitignore* is a special file stored in the root of the repository that allows a developer to explicitly exclude certain file name patterns, file extensions, and subdirectories from tracking. Any paths, files, and patterns added to this path are excluded from any `git add` operations unless you use `git add -f [fullfilepath]`. The recommended use of this file is to exclude any binaries, generated code, compile time dependencies, etc that are brought in by reproducible build processes. Here is the default [Visual Studio .gitignore](https://github.com/gundermanc/blog/blob/topic/gitin30seconds/.gitignore) which excludes subdirectories like *bin* and *obj* from tracking because they are created by the build process.

### Staged vs. Not Staged

Staged files are tracked files with changes that have been marked for inclusion into the next commit. If you run `git commit` on a repository with changes, these changes are not recorded in the Git history unless you first explicitly mark these files for inclusion via `git add [filename]`, `git add .` to stage all non-ignored files, or you instead run `git commit -a` to instruct Git to commit all changes to all *tracked files*.

**NOTE**: Deleting a file and then running `git commit` does not delete the file from tracking unless you explicitly `git add [filename]` the deleted file or run `git commit -a`. This is because a deletion of a file is still a change that needs to be staged, just like any other.

### *CRAWL* - Single branch cheat sheet
- Create a new repository:
  - Install a Git client
  - Open a Git Command Prompt or Bash session
  - `cd` to the desired directory
  - `mkdir [reponame]` to create a new repository folder
  - `cd` into the repository folder
  - Run `git init` in the root of the directory to initialize your git repository
- Check 'where' we are:
  - `git log` - The top commit is your currently checked out commit. Lists in reverse chronological order the commits upon which this commit was based. In other words, it's essentially a timeline of changes that have been made leading up to the currently checked out commit.
  - `git branch` - Tells which branch we have checked out. If you see 'detached HEAD' or a long commit hash, you are not on a commit that is tracked by a branch. More on this later... For now, you should just see master.
- Check which files are not yet tracked, or have uncommitted changes:
  - `git status` - Tells the list of untracked files, and the list of files with uncommitted changes
  - `git diff` - Shows a diff of all unstaged changes to currently tracked files. **NOTE**: this does not show changes to new files, newly tracked files (files `git add`ed and not yet committed for the first time, or changes to untracked files.
- Start or stop tracking or stage file changes:
  - `git add [filename]` - Performs two essential functions:
    - Starts tracking files in Git
    - Stages tracked files changes to be part of the next commit. e.g.: Running `git commit` only commits files previously staged with `git add`, unless you commit with `git commit -a`.
  - `git reset [filename]` - Unstages uncommitted changes to tracked files, or stops tracking a file that is staged for tracking, but not yet tracked.
- Commit changes:
  - `git commit` - Commit staged changes and/or start tracking new staged files
  - `git commit -a` - Commit changes to all tracked files (files previous marked with `git add [filename]` or `git add .`).
- Undo past changes:
  - `git revert [commitID]` - Run this on the long, unique commit ID for the desired commit in the `git log`. This will create a new commit and update all *tracked* files in the working directory to not have any of the changes from the reverted commit.
- Compare code at points in time:
  - `git diff [commitA] [commitB]` - Performs a diff of the code, showing insertions and deletions that occurred between commitA and commitB.

## *WALK* - Branching, merging, and remotes

Having a timeline of historical changes to your code is useful as an individual developer, but the real power of Git comes out in its ability to branch, re-merge, and update remote repositories by autonomously merging your changes with others.

### *WALK* - Branches

Git branches increase the overall usefulness of Git by allowing your 'timeline' of commits to diverge from a flat line into a tree of commits. Using this powerful branching functionality, individuals can essentially have snapshots of multiple divergent 'timelines' of their code, and can experiment with different features on each, whilst maintaining a clean 'master' branch for actual distribution (shipping to customers). When they have iterated enough on their code and they are happy with the experimental version, they can 'merge' these changes into the 'master' branch to update their shipping code. This allows developers to be agile and quickly drop and pickup tasks as they come up, or quickly switch to another task if they get blocked, all without losing their current progress.

By default, most repos start on the 'master' branch. To create a new branch in your respository based off your **currently checkedout commit** and to immediately switch to that branch, you can use `git checkout -b [branchname]`. Now that you're on a different branch, you can commit as you wish to this new branch, secure in the knowledge that your 'master' branch will not be affected. To switch back to your other branch at any time, you can run `git checkout [branchname]` and your working directory tracked files are reverted to the state they were in on your latest commit on that branch. Untracked files are not modified.

**NOTE:** Before switching branches I highly recommend committing whatever uncommitted changes you have, otherwise, you run the risk of them following you around and getting committed accidentally to the wrong place. I observe beginners getting frustrated with this quite often.

**NOTE:** If you encounter a message like "untracked file would be modified by switch to branch 'foo'", it's likely that you have a file that is untracked in your current branch, but is tracked in the other branch, and Git is cleverly telling you it will get overwritten if you change branches. In this case, I'd commit the file first, or delete it if you don't care about its contents, so that Git can do its business.

When you're ready to bring your diverged changes back into another branch (such as your shipping 'master' branch), checkout the branch that will *receive the changes*, and then run `git merge [sourcebranch]` to merge the commits from the source branch into the destination branch. Git will also create a 'merge commit' with any conflict resolutions that needed to occur.

### *WALK* - Merge Conflicts

Merge conflicts occur when a file is updated in two separate branches and then the branches are merged. Git is unfortunately a bit naive, so it asks for help by signaling that there is a 'merge' conflict. When this occurs, your workflow as a user is to perform the merge locally, then run `git status`, find the files with conflicts (usually listed in red and annotated with 'both modified' or similar), and open the files one by one, resolving the conflicts by hand.

Git marks merge conflicts in files by wrapping the code in `<<<<<<<<<<< CODEA ====== CODEB >>>>>>>>>>>`, where CODEA is the code from one branch, and CODEB is the code from the other branch. To resolve these by hand, simply evaluate the code, correctly combine the changes from CODEA and CODEB, and then delete the merge markers, run `git add [filename]` on the conflicting file, and then `git commit` to complete the merge and commit your conflict resolution as the merge commit

### *WALK* - Remotes

Now that we have the concept of branches in Git, allowing us to work on several tasks simultaneously, lets expand this concept to decentralized collaboratation. Enter the remote: Git remotes are a one line entry in the repository metadata that indicates a 'remote' repository from which commits can be pushed or pulled.

To create a local repository from a shared repository on the web, you first need to create an SSH key or setup another authentication mechanism. If you are using Windows and Visual Studio Team Services Git VC, you can use the [Git Credential Manager](https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases) to automatically authenticate with the server via Active Directory if you are on a Domain Joined machine. For more info on this, speak to your IT dept. If you are intending to use a Unix style Git environment or GitHub, you probably want to create a public private SSH key pair and associate it with the remote service. To do this, please see [GitHub's Docs](https://help.github.com/articles/connecting-to-github-with-ssh/).

Once you have your auth configured, you can perform the following actions:

- `git remote` - Lists all currently registered remotes for the repository.
- `git remote add [name] [repo url]` - Adds a new remote with the designated name and URL.
- `git remote rm [name]` - Deletes a remote with the designated name.
- `git clone [repo url]` - Creates a local copy of the remote repository and automatically sets up a remote called 'origin' to point to the old repo.
- `git push [remotename] [branchname]` - Attempts to push all new commits from the current branch to the remote repo and branch. This will fail if there is no common history between the repos or if the remote branch has newer commits. If the remote branch has newer commits, you must first pull, which will perform a merge of the remote branch into your local branch.
- `git push --set-upstream [remotename] [branchname]` - Attempts to push all new commits from the current branch to the remote repo and branch and sets the local branch to 'track' the remote branch. e.g.: all subsequent pushes and pulls that don't provide a remote/branch will push/pull from [remotename] [branchname]
- `git push` - Pushes to the tracking repo and remote branch, or fails if there is not one set up.
- `git pull` - Pulls from the tracking repo and remote branch, or fails if there is not one set up.
- `git fetch` - Updates the local repository will records of all branches, tags, etc. from the remote repository to this one.
- `git branch -r` - Lists all branches that exist in the **remote** repository.
- `git checkout -b [localbranch] -t [remotename/remotebranch]` - Creates a local branch that tracks a remote branch.

**NOTE:** Branches in your local repository are not the same exact branch as the branch in the remote repository. The optimum workflow for Git is to create a **local** branch that **tracks** a remote branch (e.g.: each push pushes from your local [branchname] to origin/[branchname]). **Do not attempt to work directly against remote branches, such as origin/***

### *WALK* - Working collaboratively

Working collaboratively entails using remotes to push and pull others' changes, usually from a central repository, though you can technically pull from anyone's. My recommendation is to use the shared repository model. In doing so, the following are useful highlevel workflows:

- Clone a respository:
  - `git clone [repo url]` - Creates a local copy of the remote repository and automatically sets up a remote called 'origin' to point to the old repo.
- Create a local 'version' of a remote branch:
  - `git checkout -b [branch] -t [remotename/branch]` - Creates a local branch of the same name tracking the remote branch
- Pushing and pulling from a remote branch:
  - `git push` - Pushes to the tracking repo and remote branch, or fails if there is not one set up.
  - `git pull` - Pulls from the tracking repo and remote branch, or fails if there is not one set up.
- Create a feature/topic branch:
  - `git checkout -b [branchname]` - Creates a new local branch named 'branchname'
  - `git push --set-upstream [remotename] [branchname]` - Creates a new remote branch (or pushes to the existing one, if there is one) and starts tracking it

## *RUN* - Refs, Tags, Rebase, Reflog, Submodules

### *RUN* - Tags

Tags are special way of marking a particular commit with a name. Unlike branches, tags are static and always point to the same commit, making them ideal for marking commits for build numbers and release versions. You can add tags with `git tag [name]` or view them with `git tag`.

### *RUN* - Refs

There is a common misconception by users that a branch in Git is a timeline of commits going back in time and that a branch is a container that 'holds' their commits. The reality is quite a bit simpler, commits in Git exist largely independently of the branching and tagging system, and branches and tags are both slightly different cases of something called a 'ref'. A 'ref' is simply a pointer that holds a commit ID that points to a particular commit.

Because of this pointer-like aspect of refs, there are some interesting implications:

- Branches: while checked out to a specific branch, on commit, the branch ref is updated to point at the new commit, meaning that branches 'follow' your new commits.
- Tags: unlike branches, regardless of which commit, tag, or branch you currently have checked out, committing does not update the tag refs.
- 'HEAD': your currently checked out commit or branch. You can use this as a reference point, e.g., by `git checkout HEAD~2` to go back another 2 commits
- 'detached HEAD': your currently checked out commit is not on a branch. Believe it or not, you can actually still commit in this state. Even though your new commits aren't tracked by a branch, you can retroactively `git checkout -b [branchname]` to create branch that is tracking these commits, or even find these commits later on and create a branch tracking them after switching to another branch by looking at your `git reflog`
- You can (very dangerously) throw away all of your changes and update the pointer for a branch to point at a specific commit by running `git reset --hard [commmitID]`, or even update your branch to point to a specific commit, liquidating all unneeded commits, and adding them as uncommitted changes to your working directory via `git reset --soft [commitID]`
- You can (very dangerously) set one branch to point at the *exact same commits* as another or sync it to a build tag using `git reset --hard [commitID]`

### *RUN* - Rebase

`git rebase -i [commitID]` is my secret to smoothing my squiggly thought process into a more logical, linear flow line. Running `git rebase -i HEAD~4` for example, opens VIM with a list of the last four commits, and gives you the option to rewrite your Git commit history with 'drop' (delete), 'pick' (keep), 'fixup' (squash into the previous), or 'reword' (changing the commit message) of each commit. Upon closing VIM, Git deletes the commits since commitID, and then one by one, creates new commits by effectively performing a `git cherry-pick [commitID]` of the non-dropped commits onto the branch. In the end, each of the non-dropped commits is brand new and has a new commit ID.

**NOTE**: because rebase is rewriting history, although the files can be identical, you have broken the Git history for anyone that has already based their branch off of the rebased commits, meaning that at merge time, you will be unable to merge. My recommendation is to never rebase commits once you have pushed them to a remote.

- `git rebase -i [commitID]` - dissolves and creates new commits derived from all commits between HEAD and commitID with the specified change to the history.
- `git cherry-pick [commitID]` - creates a new commit with a distinct commit ID on the current branch by copying the changes from the specified commitID.
- `git commit --amend` - creates a new commit from the combination of your previous commit and your uncommitted changes.

### *RUN* - Reflog

`git reflog` is the tool of last resort (and my personal favorite) for finding lost commits and recovering from bad merges, rebases, and committing in a headless state. If you ever find yourself in a bad state, don't know where you lost your work, and need to get back to where you were, you can look at the output of reflog to see the last 20-30 commits that were your repositories HEAD, as well as the transitions you took to get to them. Using this, you can find the last known good commit, and then either `git checkout [commitID]` to go there headless, or `git reset --hard [commitID]` to force update your current branch's ref to point to that commit.

**NOTE**: This history only goes back so far, so if you find yourself in a bad state, try to minimize your usage of `git checkout [commitID]` until you are back in a known good state.

### *RUN* - Submodules

Submodules are complete Git repositories, nested within a larger repository. Using Git submodules encourages componentization and code reuse by allowing multiple consumers to pull the same dependencies in in source code form.

**NOTE:** Because submodules are complete Git repositories, they come with all of the implications of a Git repository. The submodule is checked out to a particular commit, so, any commits made to the submodule are not pushed upstream unless the developer explicitly submits a PR against the upstream repository with the change.

Submodules are a complex area which require a lot of decision-making to use effectively, so, I defer this to the official [Git book](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
