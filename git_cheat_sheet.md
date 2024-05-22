### Reference: https://github.com/internetarchive/openlibrary

# Table of Contents

- [Forking and Cloning the Open Library Repository](#forking-and-cloning-the-open-library-repository)
- [Working on Your Branch](#working-on-your-branch)
- [Making Changes and Creating a Pull Request](#making-changes-and-creating-a-pull-request)
- [Updating Your Pull Request](#updating-your-pull-request)
- [Troubleshooting Your Pull Request](#troubleshooting-your-pull-request)
- [Commit History Manipulation](#commit-history-manipulation)
- [Pre-commit and the GitHub CI](#pre-commit-and-the-github-ci)

## Forking and Cloning the Open Library Repository

### Fork the Open Library repository

Fork the Open Library repository using the GitHub UI by logging in to GitHub, going to [https://github.com/internetarchive/openlibrary](https://github.com/internetarchive/openlibrary) and clicking the Fork button in the upper right corner:
![GitHub Fork](https://archive.org/download/screenshot20191211at11.12.56/fork.jpg)

### Clone the forked repository on to your local computer

This creates a local copy of your own fork of the Open Library repository, in a directory called *openlibrary*. Your fork on the GitHub servers is a remote called *origin*. By default, you are looking at the *master* branch.

Make sure you `git clone` openlibrary using `ssh` instead of `https` as git submodules (e.g. `infogami` and `acs`) may not fetch correctly otherwise.

```sh
git clone git@github.com:USERNAME/openlibrary.git
```

#### Permission denied while cloning

If you have not added your public SSH key to GitHub you may see:

```sh
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

To fix this, first [generate a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) if you have not already done so, and then [add the SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).

#### Modifying a repository wrongly cloned with `https` (or one that is missing the infogami module)
You can modify an existing openlibrary repository that was inadvertently cloned with `https` by running:
```sh
git remote rm origin
git remote add origin git@github.com:USERNAME/openlibrary.git
git submodule init; git submodule sync; git submodule update
```

#### Fix line endings, symlinks and git submodules (only for Windows users not using a Linux VM)

Here, the project files need LF line endings because they are used in a Linux Docker container, even if run from Windows. Additionally, symlinks don't clone properly, and this creates issues for the git submodules, among other things.

For more on git and line endings, see [Configuring Git to handle line endings](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings).

**Note: if you get permission issues while executing these commands please run git the bash shell as an Administrator.**

```sh
# Get in the project root directory
cd openlibrary

# Configure Git to keep LF line endings on checkout even on Windows.
git config core.autocrlf input

# Enable symlinks
git config core.symlinks true

# Build submodules
git submodule init; git submodule sync; git submodule update

# Stage indexed files for removal so git reset updates them
git rm --cached -r .  # Don't forget the "."

# Reset the repo (removes any changes you've made to files and is likely to give an error if not administrator)
git reset --hard      # You will almost certainly need to use git-bash as administrator for this.
```

### Add 'upstream' repo to list of remotes

```sh
cd openlibrary
git remote add upstream https://github.com/internetarchive/openlibrary.git
```

### Verify the new remote named 'upstream'

```sh
$ git remote -v
origin  git@github.com:USERNAME/openlibrary.git (fetch)
origin  git@github.com:USERNAME/openlibrary.git (push)
upstream        https://github.com/internetarchive/openlibrary.git (fetch)
upstream        https://github.com/internetarchive/openlibrary.git (push)
```
Note that `origin` is `git@`. If it is not, see [Forking and Cloning the Open Library Repository](#forking-and-cloning-the-open-library-repository).

## Working on Your Branch
Because new commits are frequently merged into the Open Library repository, it's important to make sure that the branch you're working on is up to date with the upstream `master` version.

Before creating a new branch and **each time** you start work on an existing branch, you should run the following commands to make sure your master branch is up to date:
```
git switch master
git git pull --ff-only upstream master
git push origin master
```
**Note:** When running the `pull --ff-only`, you may see the error `fatal: Not possible to fast-forward`. If so, see [Out-of-Sync Branches](#out-of-sync-branches) for more instructions re: getting your branch up to date.

You can then create a new branch for your issue:
```
git switch -c 1234/fix/fix-the-thing
```

Or, if you are returning to work on a previously created branch, rebase* with master:
```
git switch existing-branch-name
git rebase master
```
\***Note:** Rebasing is the equivalent of "lifting" all the commits in your branch, and placing them on top of the latest master. It effectively changes the *base* of your branch/commits. If you encounter any errors in the rebase process, see the [troubleshooting guide](#troubleshooting-your-pull-request).

You can then confirm that everything is up to date by running `git status` and/or visiting your remote master branch at `github.com/your-username/openlibrary`, where you should see the following:
<img width="777" alt="OL_Git_UpdatedMaster" src="https://github.com/internetarchive/openlibrary/assets/79802377/1c47c7dd-d56e-4098-8924-8689bd91b8a1">

If everything looks good, you can continue following the steps in [Making Changes and Creating a Pull Request](#making-changes-and-creating-a-pull-request) to commit, test and submit your changes.

If not, read on!

### Out-of-Sync Branches

Your master or working branch may get out-of-sync. In general, **do not use VSCODE or GITHUB MERGE** to resolve merge conflicts. Here are some commands to run for common out-of-sync situations:

(Note: You check the status of your branch by running `git status` or going to `github.com/your-username/openlibrary`)

**Master is behind upstream master**
![OL_Git_UnsyncedMaster](https://github.com/internetarchive/openlibrary/assets/79802377/80fe990e-72be-4eff-beb2-b4ad7f888378)

```
git switch master
git pull upstream master
git push origin master
```
**Master is behind and ahead of upstream master**
![OL_Git_UnsyncedMaster2](https://github.com/internetarchive/openlibrary/assets/79802377/a28aa6f0-2136-4bad-8564-bea9b893c44d)

```
git switch master
git reset --hard HEAD~[the number of commits you are ahead by]
git pull --ff-only upstream master
git push -f origin master
```
**Note:** A "hard" `reset` will permanently undo your changes, so make sure you're on the master branch before resetting. It's always safe to hard reset your master branch because it is supposed to be identical to the upstream version. 

This means that if you've accidentally committed something to your master branch, you can easily `reset` as many commits as you'd like -- often `git reset --hard HEAD~100` is a good way to get a clean slate -- then just `pull` in the upstream version to remain up to date.

**Working Branch is behind upstream master (it will always be ahead if you've made changes):**
![OL_Git_UnsyncedBranch](https://github.com/internetarchive/openlibrary/assets/79802377/2074f643-1bf0-45aa-ab10-cb68e9763e78)

```
git switch master
git pull --ff-only upstream master
git push origin master
git switch your-branch-name
git rebase master
```
If rebasing your branch fails or provokes merge conflicts, see [Troubleshooting Your Pull Request](#troubleshooting-your-pull-request). 

When you're done working and try to push your branch with `git push origin HEAD`, you may encounter the following error:

<img width="571" alt="OL_Failed_Push" src="https://github.com/internetarchive/openlibrary/assets/140550988/8eaa623b-eff9-4d81-8099-e40bfea732de">

This usually means that, as a result of the rebase, you'll need to force push* your updates:
```
git push -f origin HEAD
```

\***Note:** While regular pushing just adds new commits to the remote branch, force pushing _replaces_ the commits on the remote branch with the commits on your local branch, effectively re-writing the remote commit history. Sometimes when you perform a rebase, you will have to force push to your branch. 

If so, be sure to **force push with care:** You should _only_ force push if working on one of your own branches. If working on a branch which other people are also pushing to, force pushing is dangerous because it can override others' work. In that case, use `--force-with-lease`; this will force push _only_ if someone else hasn't made any changes to the branch. 

If none of the above solved the issue, you can consult a staff member or the issue's lead, and/or see [Troubleshooting your Pull Request](#troubleshooting-your-pull-request).

## Making Changes and Creating a Pull Request
**1. Make sure master is up-to-date:**
```sh
git switch master
git pull --ff-only upstream master
git push origin master
```
For troubleshooting or to learn more, see [working on your branch](#working-on-your-branch).

**2. [Create a new branch for the feature or issue you plan to work on](https://github.com/internetarchive/openlibrary/blob/master/CONTRIBUTING.md#development-practices) and switch to it:**
```
git switch -c 1234/fix/fix-the-thing
```
Specifying `-c` creates a new branch, and `switch` switches to it. The typical branch name format is `[issue number]/[fix/feature/hotfix]/[short-issue-description]`

**3. Make changes/commit:**
```
git add the-file.html
git commit -m "Fixed the thing"
```
A commit message should answer three primary questions;
- Why is this change necessary?
- How does this commit address the issue?
- What effects does this change have?

**4. Test your changes:**
```sh
docker compose run --rm home make test
```
When you submit your pull request, the [GitHub CI server](#pre-commit-and-the-github-ci) will automatically run a few more tests and formatting checks. 

If you'd like, you can run these checks before you submit by [installing `pre-commit` locally](#running-pre-commit-locally-recommended), or run a [one-off formatting check](https://github.com/internetarchive/openlibrary/wiki/Testing#linting). 

**5. Push the branch:**
```
git push origin HEAD
```
**Note:** `HEAD` refers to your current branch, so make sure you're on the right branch.

**6. Submit your pull request:**

Go to [github.com/internetarchive/openlibrary/pulls](https://github.com/internetarchive/openlibrary/pulls); find the message at the top re: your branch and click `Compare & pull request`.
![OL_Git_PR](https://github.com/internetarchive/openlibrary/assets/79802377/d58c3d92-e281-4775-9eab-084490887d11)

Confirm that the changes in `Files Changed` match the changes you have made on your working branch and intend for this PR. If not, see [Troubleshooting Your Pull Request](#troubleshooting-your-pull-request).

If everything looks right, you can write out an explanation of your changes using the provided template and submit. Your code is now ready for review!

**Reminder:** Any time you return to your branch to make changes, be sure to follow steps in [Working on Your Branch](#working-on-your-branch) to make sure your branch stays up to date.

## Updating your Pull Request

Pull requests often receive feedback; to make [requested changes](https://github.com/internetarchive/openlibrary/blob/master/CONTRIBUTING.md#submitting-pull-requests) to your existing pull request:

**1. Make sure your branch is up to date with master:** 
```
git switch master
git pull --ff-only upstream master
git push origin master
```
**2. Switch to your branch:**
```
git switch your-branch-name
git rebase master
```
For more on rebasing, see [Working on Your Branch](#working-on-your-branch).

**3. Make your changes and commit (same as step 3 above)**

**4. Push your changes up:**
```
git push origin HEAD
```
**Note:** When trying to push you may see a warning like the following:

<img width="571" alt="OL_Failed_Push" src="https://github.com/internetarchive/openlibrary/assets/140550988/4f546ed0-02ed-4f34-8b98-91361e660768">

This usually means you need to _force-push_ your changes as a result of the rebase:
```
git push -f origin HEAD
```
To learn more, see [Working on Your Branch](#working-on-your-branch).
  
## Troubleshooting Your Pull Request
**Note:** If you encounter any errors you don't understand and/or don't want to try working through this troubleshooting guide yourself, feel free to reach out to the issue's lead, who can best case guide you to the correct resources or worst case just make any necessary fixes on your remote branch for you.

**Tips for what to do in common situations, such as:**
- [Rebase Fails with Merge Conflict Error](#rebase-fails-with-merge-conflict-error)
- [PR Includes Unrelated Commits](#pr-includes-unrelated-commits)
- [Commits Include Unrelated Changes](#commits-include-unrelated-changes) 
- [Failing an Automated GitHub CI Check](#the-github-ci-server)
- [Failing a Local Pre-Commit Check](#running-pre-commit-locally-recommended)
- [Failing the `Generate POT` Check](#failing-the-generate-pot-check)
- [Could Not Read From Remote Repository](#could-not-read-from-remote-repository)
- [Manual Merge Conflict Resolution](#manual-merge-conflict-resolution)

### Rebase Fails With Merge Conflict Error
Sometimes when you try to `rebase` your branch after [updating your master branch](#working-on-your-branch), you'll get an error message like this:

```
Auto-merging openlibrary/templates/about/team.json
CONFLICT (content): Merge conflict in openlibrary/templates/about/team.json
error: could not apply 447122b8d... Switch out personal URL for team page
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

There is a fairly simple way to resolve a conflict like this in VSCode's editor, **but** you first want to make 100% sure that you're dealing with an actual [merge conflict](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/about-merge-conflicts), as this error can sometimes happen as a result of accidental commits on one of your branches or another out-of-date branch issue. 

If this is the case, using the merge conflict resolution tools in VSCode or GitHub will only create more problems, so before starting a manual resolution, you'll want to run:
```
git rebase --abort
```
And then check in with your issue's lead to determine what steps to follow and/ or double-check to ensure you're dealing with an actual merge conflict by:
1. Following the steps in [Working on Your Branch](#working-on-your-branch) to confirm that your master is up to date and not "ahead" by any commits before trying to rebase again.
2. Ensuring that your PR does not include any unrelated commits, by checking the "Outgoing" commits in the VSCode Source Control tab and/or the "Commits" tab on your PR on GitHub. If you find any, follow the steps in [PR Includes Unrelated Commits](#pr-includes-unrelated-commits).
3. Ensuring that your commits don't include any unrelated changes, by checking the "Outgoing" changes in the VSCode Source Control tab and/or the "Files changed" tab on your PR on GitHub. If you find any, follow the steps in [Commits Include Unrelated Changes](#commits-include-unrelated-changes).
4. Ensuring that you aren't getting this conflict because the `pre-commit` CI made some commits on your behalf. You'll see these in the "Commits" section on GitHub, and you'll want to pull them into your branch with `git pull upstream name-of-your-branch` before trying to rebase. After this, you'll need to run a `git push -f origin HEAD` to keep everything up to date.

If you've tried each of the above steps, and you're still getting the merge conflict error, you can now either contact your issue's lead for help moving forward, or begin to [resolve it manually](#manual-merge-conflict-resolution).

### PR Includes Unrelated Commits
Sometimes if you have a look at the outgoing changes in the VSCode Source Control tab or the "Commits" in your submitted PR, you'll notice that there are other changes included along with yours that either a) you made but didn't intend to include in this PR, or b) were made by other people.

The most common reason this would happen is that you pulled in the upstream changes but forgot to push to your remote branch as well. So before proceeding, you'll want to confirm that you've pushed everything up:
```
git switch master
git pull --ff-only upstream master
git push origin master
git switch your-branch
git push -f origin HEAD
```

If you can see that the extra commits are now gone, you're good to go. If not, this means that you'll need to manually remove the unneeded commits from your branch, like so:

1. Switch to the correct branch and run the following command. If using VSCode, it's recommended that you do this in the [built-in terminal](https://code.visualstudio.com/docs/terminal/basics) to ensure that the next few steps also happen in VSCode. 
```
git rebase -i master
```
This will open a text editor that you can use to select which commits you actually want included in your PR, i.e.:
```
pick eb8ab51 [Your commit message]
pick a18d382 [Someone else's commit you don't want]
pick 76b9883 [Your commit message]

# Rebase ef7d551..23961be onto ef7d551 (7 commands)
```
To remove an unwanted commit, simply switch the text from `pick` to `drop` in the text editor and close the window. Once this is done, you can double-check that the unwanted commits are now gone, and force-push your changes:
```
git push -f origin HEAD
```
**Note:** If the text editor opens in something other than VSCode and you're unsure how to close it, and/or you'd like to try some more advanced commit manipulation methods, see [Commit History Manipulation](#commit-history-manipulation) to learn more.

### Commits Include Unrelated Changes
Sometimes if you have a look at the outgoing changes in the VSCode Source Control tab or the "Files changed" in your submitted PR, you'll notice that there are a number of changes made and/or files changed that you didn't intend to include in the PR. 

If you look at the commit history ("Commits" tab on GitHub) and can confirm that those changes each come from someone else's commit or a separate commit of yours that was accidentally included, you can follow the steps in [PR Includes Unrelated Commits](#pr-includes-unrelated-commits). 

But if you can see that the unrelated changes are actually included in commits you do want to keep, you can do the following:
1. Ensure your master branch is up to date:
```
git switch master
git pull --ff-only upstream master
git push origin master
git switch your-branch
``` 

2. "Soft" reset as many commits as you need, i.e.:
```
git reset --soft HEAD~[number of commits to undo]
```
This will effectively undo your commit (or commits) and return the changes to staging. You can then undo any changes you don't want included, i.e.:

In the VSCode Source Control tab:
- Hover over the changed you file you want to undo changes to
- Hit the minus sign to remove it from staging
- Hit the reverse symbol to undo the changes

Or, in the terminal:
```
# Remove unwanted file from staging -- or use a . instead of filename to unstage all
git restore --staged path/to/your/file

# Undo changes to selected file
git checkout -- path/to/your/file
```
You can then re-commit your desired changes, and push your changes back up:

```
# Add any files you want to commit back to staging if needed
git add file-to-include
git commit -m "Your original commit message"

# Force push to overwrite the remote version
git push -f origin HEAD
```

### Failing the `Generate POT` check
If your commit involves adding, removing or altering text that will be visible to the user and is properly internationalized, an update of the translation template file will be automatically bundled in with your changes via `pre-commit`. 

To learn more, see [Pre-commit and the GitHub CI](#pre-commit-and-the-github-ci).

**What this means:**

If you're running `pre-commit` locally:
- Your code will "fail" a test called `Generate POT`, give you the error message `Files were modified by this hook`, and add `messages.pot` changes to your git unstaged changes.
- All you need to do to "pass" the test is add the `messages.pot` file to staging and redo your commit; the `Generate POT` test should now pass, and your changes will be immediately available to translators once your branch is merged. 

If you're not running `pre-commit` locally:
- Your code will "fail" the `pre-commit` check run by the GitHub Continuous Integration (CI) server
- The CI will then push a new commit to your remote branch that contains the necessary `messages.pot` updates and now passes the `pre-commit` check 
- You don't need to do anything else after this, but if you want to make and push further changes to the PR, it would be wise to first run a `git pull origin HEAD` to pull in the new `messages.pot` changes and avoid conflicts in future pushes

### Could Not Read from Remote Repository
It may happen that when you try to pull in the `upstream` version of the repository, you'll get the following error:
```
fatal: 'upstream' does not appear to be a git repository
fatal: Could not read from remote repository

Please make sure you have the correct access rights and the repository exists.
```
This just means that your branch has accidentally gotten disconnected from the OL master branch. All you need to do to fix it is [add `upstream` repo to list of remotes](#add-upstream-repo-to-list-of-remotes) and [double-check that it worked](#verify-the-new-remote-named-upstream). 

You can then safely try pulling again to keep everything up to date.

### Manual Merge Conflict Resolution 
**Note:** Manual merge conflict resolution can get a little tricky, so if at any time you want to stop and ask for input from your issue's lead, you can simply run `git rebase --abort` to return everything to its pre-rebase state. The lead will also be able to edit your branch for you on GitHub to resolve the conflict if necessary.

A [merge conflict](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/about-merge-conflicts) happens when your changes conflict with other changes that have just been added to the repository.

For instance if you changed:
```
<div>Hello world!</div>
```
to
```
<p>Hello world!</p>
```

And then you pulled in someone else's change from `upstream` that had for instance changed `Hello world!` to `Hi world!`, you would be faced with a merge conflict, because git would not know whether to make the line `<p>Hi world!</p>` (both changes combined) or treat your version (`<p>Hello world!</p>`) or their version (`<div>Hi World!</div>`) as authoritative.

Similarly, if someone else had made conflicting changes but you hadn't rebased to include them before submitting your PR, GitHub would add a warning to the PR that your branch could not be merged until conflicts were resolved.

**Note:** You'll only want to do this if you're 100% sure you're dealing with a real merge conflict, i.e. you can see a recent commit to the codebase that would conflict with one of your commits. To double-check and confirm you've got a true merge conflict, see [Rebase Fails with Merge Conflict Error](#rebase-fails-with-merge-conflict-error).

Once you've ensured you do have a merge conflict, you can start the `rebase` again in VSCode by switching to your branch and running `git rebase master` and use its built-in merge editor to resolve the conflict(s):

![Merge Conflict Editor](https://github.com/internetarchive/openlibrary/assets/140550988/a9a6a5fa-f850-43c3-819e-b9014031012a)

Conflicting changes will be highlighted, and you'll have three main options:
1. "Accept Current Change" - Line becomes `<p>Hello world!</p>`, your commit now overwrites theirs, and includes changing "Hi world!" back to "Hello world!"
2. "Accept Incoming" - Line becomes `<div>Hi world!</div>`, you undo all your own changes to the line and keep it as they have it
3. Custom/Combination -- 
- To use a combination of the two changes, i.e. `<p>Hi world!</p>`, select "Resolve in Merge Editor" which will open this view:
![Full Merge Editor](https://github.com/internetarchive/openlibrary/assets/140550988/b04f8f34-dfd3-4c7d-abf6-bd82405558df)
- Either select "Accept Combination" or edit the result text directly to match the desired combination
- Select "Complete Merge"
- You'll see your resulting change ready to go in the source control tab, with your previous commit message already filled in, and you can just hit "Continue" to re-commit and finish up 

Once you're done rebasing, you'll want to force-push your changes up using:
```
git push -f origin HEAD
```
And then congratulate yourself on making it through a merge conflict resolution!

## Commit History Manipulation

**WARNING:** You can cause yourself some headaches with this feature! But it's easily one of the most powerful things about using `git`, so it's worth learning :)

Sometimes you'll want to rearrange/reword/combine commits to keep the history neat. To do this, on your branch, run:

```sh
git rebase -i master
```

| Info |
| --- |
| The `-i` is for interactive. The command is also specified often as something like `git rebase -i HEAD~2`. `HEAD` refers to the current, latest commit in your branch; `~2` goes back 2 in the history, so you'll be manipulating the last 2 commits. `git rebase -i master` lets you manipulate all the commits on your branch. |

| Info |
| --- |
| By default, `git` will open up an editor in your terminal (likely `vim`). If you would rather use VS Code, run `git config --global core.editor "code"` once, and then `git` will always use VS Code when prompting for a rebase, or a commit message. |
| If you happen to find yourself stuck in `vim` and don't know how to get out, press `ggdG:wq` (in order: `g` for "go to", `g` for "top of file", `d` for delete, `G` for "to bottom of file". So `ggdG` is for "go to the top of the file, and delete everything". This is how you cancel `git rebase -i`. Then: `:` for "enter command-line mode", `w` for "save", `q` for "quit". So `wq` is "save and quit". If you're interested in learning more about `vim`, see https://vim.fandom.com/wiki/Tutorial |

This will open a text editor, and let you edit all the commits that your branch has. It will look something like this:

```
pick eb8ab51 Made footer's HTML translatable
pick a18d382 Fixed some typos
pick 76b9883 Made footer's HTML translatable: added the missing translatable strings
pick 73c78b7 Fix git hash version i18n
pick 377e121 Added a missing translatable string
pick caf9507 Reverted unrelated changes to the PR
pick 23961be Clean up trailing whitespace

# Rebase ef7d551..23961be onto ef7d551 (7 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
```

If you decide you want to cancel the rebase, delete everything, and then save. That tells `git` to do nothing.

To continue with the rebase, save the file. `git` will then replay all the instructions/commits in that file. If there is a conflict, it will pause to let you fix them. See [Manual Merge Conflict Resolution](#manual-merge-conflict-resolution).

## Pre-commit and the GitHub CI
To automatically ensure that certain formatting practices are maintained throughout the codebase, and that any incoming pull requests pass a set of requisite JavaScript and Python checks, Open Library uses GitHub's Continuous Integration (CI) Server with a set of [`pre-commit` hooks](https://pre-commit.com/) to run a series of automated checks on incoming PRs.

### The GitHub CI Server
If you don't have `pre-commit` installed locally and your PR fails any one of the checks, there are two things that may happen when you push your changes:
1. One of the checks initially fails, but then the `pre-commit` bot pushes a new commit for you that fixes the problem. If this is the case, you'll see a commit that looks like this:

<img width="782" alt="Pre-commit bot updates" src="https://github.com/internetarchive/openlibrary/assets/140550988/444c8708-e598-4562-babb-58ae2c49d2d2">

This means that the `pre-commit` bot auto-fixed the problem for you, which is very common in the case of simple formatting errors. In this case, you're all set, but if you plan on making any more changes to your branch, it's a good idea to pull in `pre-commit`'s changes with `git pull origin HEAD` to ensure you don't run into any conflicts.

2. The check simply fails and you'll see something that looks like this:

<img width="757" alt="Failing pre-commit check" src="https://github.com/internetarchive/openlibrary/assets/140550988/882f228a-a7dd-4385-81a6-a70895afe7b8">

This means that the problem that the CI identified requires human intervention, which means you'll want to click "Details" to see what exactly tripped up the test, try to fix the problem locally, and push up a new solution with `git push origin HEAD`. If you're not sure what is causing the error or how to fix it, this is a great time to reach out to the issue's lead for guidance on how to proceed.

### Running `pre-commit` Locally (Recommended)
To test everything the CI server checks (JavaScript, `mypy`, `black`, `ruff`, etc.), and to do so automatically at the time of commit, one can run, in the local environment, outside of Docker, a Python program named `pre-commit`. This will use `git`'s [hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to run Open Library-specific linting checks when committing code in `git`. Because `pre-commit` integrates with `git`, that means it runs outside of Docker, and needs to be available to `git` in your current environment.

If you have `pre-commit` installed, the checks will run locally each time you add a commit to your branch. This way, you don't have to worry about re-pushing your changes every time you encounter a `pre-commit` error, which is especially nice in the case of simple formatting issues like trailing whitespace.

If your commit fails any of the checks, there are two things that may happen:
1. The check will initially fail and your staged changes will not be committed, but `pre-commit` will auto-add a new change that fixes the problem, in which case you'll see a new unstaged git change and something like this:

<img width="564" alt="pre-commit modifying files" src="https://github.com/internetarchive/openlibrary/assets/140550988/e0f3a63d-bd66-4057-a640-a8d751407d15">

All you need to do in this case is add the change to staging and commit again. The check should pass, and you should now be able to push your changes.

2. The check will simply fail, your staged changes will not be committed, and you'll see an error message like this:
<img width="561" alt="Failed mypy example" src="https://github.com/internetarchive/openlibrary/assets/140550988/bb5a6118-6e51-4710-9d6d-c1a542410b5e">

This means that the problem requires human intervention, which means that you can fix the problem locally, using the error message info and/or guidance from the issue's lead, and then re-commit and push your changes as needed. 

#### Installing `pre-commit`
Prerequisites:
- the version of your current Python interpreter must match the version of Python specified in the `default_language_version` section of [`.pre-commit-config.yaml`](https://github.com/internetarchive/openlibrary/blob/master/.pre-commit-config.yaml).

Although a complete discussion of managing Python's versions and Python's virtual environments is outside the scope of this discussion, it is likely worth creating a virtual environment for each Python project on which you work. See Python's own documentation about [`venv`](https://docs.python.org/3/library/venv.html) for one such approach to managing virtual environments. Additionally, if your `python3 --version` doesn't match the version specified in `.pre-commit-config.yaml`, consider [`pyenv`](https://github.com/pyenv/pyenv) on Linux, macOS, or Windows Subsystem for Linux, or [`pyenv-win`](https://github.com/pyenv-win/pyenv-win) on Windows outside of the Windows Subsystem for Linux.

*Note:* this will install a `git` commit hook that will run prior to every commit. As there are times where one may simply wish to commit code, even if it will fail the linting, **one can override commit hooks with `git commit --no-verify`**. For more on `pre-commit`, see https://pre-commit.com/.

To enable `pre-commit`, run the following in your local shell outside of Docker:
1. `pip install pre-commit` or `brew install pre-commit`; and
2. `pre-commit install`

Henceforth, `pre-commit` will lint your code with every `git commit` (unless you commit with `git commit --no-verify` to disable running the hooks). To manually run `pre-commit`, you can execute `pre-commit run --all-files`.

If you see an error similar to either of the following, please ensure you the version of you Python interpreter matches the version specified in `.pre-commit-config.yaml`:
```
An unexpected error has occurred: CalledProcessError: command: ('/home/scott/.pyenv/versions/3.9/bin/python3.9', '-mvirtualenv', '/home/scott/.cache/pre-commit/repolh5wc3hy/py_env-python3.11', '-p', 'python3.11')
return code: 1
stdout:
    RuntimeError: failed to find interpreter for Builtin discover of python_spec='python3.11'
stderr: (none)
Check the log at /home/scott/.cache/pre-commit/pre-commit.log
```

To remove `pre-commit`, run `pre-commit uninstall`.

## References
- Getting Started flow roughly based on https://gist.github.com/Chaser324/ce0505fbed06b947d962
