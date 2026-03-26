Every commit is a snapshot of the traacked files, internall stored as a delta.

Since we need to combine the deltas from the root to the current commit to get to the current state of the project,
every commit points to its previous commit.

Main is just a pointer to a specififc commit (and more generally, branches are pointers to commits too)

Since both commits and branches are light, we should commit and branch often. Branches are great to logically divide your work.

A branch (since it points to a commit), traversing that commit from that leaf all the way to point of branching and then al the way to the root will create the cuurent state of the code in this branch.

The asterisk says which branch you are currently on.

git switch -c <branch_name> to create a branch and switch to it.

At the point of divergence (typically branch creation commit), the tree splits into two, such that that creation commit now has two children.

When merging commits from different branches, you basically create a merge commit that that has two parents (the two branches)

You go to main, and then execute : git merge <branch_name>
Alternatively, you could have gone to the branch and executed git merge main


HEAD is simply a pointer to the most recent commit. It usually to the current branch (which points towards the most recenet commit) git checkout <current_commit> reveals the head pointer.

The caret operator (^) is used to append it to a branch name to refer to the parent commit of the current commit. For example, git checkout main^ will take you to the parent commit of the most recent commit on the main branch. similarly, git checkout main^^ will take you to the grandparent commit of the most recent commit on the main branch.

we can also do git checkout HEAD^ to go to the parent commit of the current commit, indefinitely to travel back in time.

A better way is to use the tilde operator (~) to provide the exact number of commits to go back. For example, git checkout main~3 will take you to the commit that is three commits before the most recent commit on the main branch.

Branch FOrcing:
We can use relative referencing with the ~ opertor to move the branch pointer to a specific commit. For example, git branch -f main HEAD~2 will move the main branch pointer to the commit that is two commits before the current commit pointed to by HEAD. This effectively "forcibly" moves the main branch to that commit, which can be useful for undoing changes or resetting the branch to a previous state. However, it should be used with caution as it can lead to loss of commits if not used properly.


Resets:
Git reset is a powerful command that allows you to undo changes and move the HEAD pointer to a specific commit. It has three main modes: soft, mixed, and hard. This is good for local changes, but not for remote changes since it can cause conflicts with other collaborators.

git reset HEAD~2 --soft : moves the HEAD pointer to the commit that is two commits before the current commit, but keeps the changes in the staging area and working directory. This allows you to make further changes or commit again.

git reset HEAD~2 --mixed : moves the HEAD pointer to the commit that is two commits before the current commit, and also unstages the changes, but keeps them in the working directory. This allows you to review the changes before staging or committing again.

git reset HEAD~2 --hard : moves the HEAD pointer to the commit that is two commits before the current commit, and also discards all changes in the staging area and working directory. This effectively resets the branch to that commit, and any changes made after that commit will be lost. This should be used with caution as it can lead to loss of work if not used properly.

Revert:
Git revert is a command that creates a new commit that undoes the changes made in a previous commit. This is a safer option for undoing changes, especially when working with remote repositories, as it does not alter the commit history. Instead of removing commits, it adds a new commit that negates the changes of a specific commit. This allows you to maintain a clear and accurate history of your.

git revert <commit_hash> : creates a new commit that undoes the changes made in the specified commit. This is useful for undoing changes without altering the commit history, especially when working with remote repositories.

Eg: git revert HEAD~2 will create a new commit that undoes the changes made in the commit that is two commits before the current commit.

Cherrypicking:
Git cherry-pick is a command that allows you to apply the changes from any set of existing commits onto your current branch. This is useful for selectively applying specific changes from one branch to another without merging the entire branch.

Eg: from main, you execute git cherry-pick C1 C2 C3 where C1, C2, and C3 are commit hashes from different branches or the same branch. This will apply the changes from those commits onto the main branch, creating new commits in the process. 

Interactive Rebasing:
Git interactive rebase is a powerful command that allows you to edit, reorder, squash, or drop commits in your branch's history. This is useful for cleaning up your commit history before merging or sharing your branch with others.

Eg: git rebase -i HEAD~5 will open an interactive editor with the last 5 commits in your branch. You can then choose to edit, reorder, squash, or drop commits as needed. This allows you to create a cleaner and more organized commit history before merging or sharing your branch with others.

Locally stacked commits:


Amending COmmits:
Git commit --amend is a command that allows you to modify the most recent commit. This is useful for correcting mistakes, adding forgotten changes, or updating the commit message.

But, if we want to make changes to an older commier, we ned to interactively rebase to bring that commit to the top and then amend it, and then rebase back to the original state. This is a bit of a hassle, but it allows you to modify older commits without altering the commit history.

But, there is another way with cherry-picking. You can cherry-pick the commit you want to modify onto a new branch, make the necessary changes, and then cherry-pick it back onto the original branch. This allows you to modify older commits without altering the commit history, and it can be more straightforward than using interactive rebase for this purpose.

Remember that git cherry-pick will plop down a commit from anywhere in the tree onto HEAD (as long as that commit isn't an ancestor of HEAD).

As you have learned from previous lessons, branches are easy to move around and often refer to different commits as work is completed on them. Branches are easily mutated, often temporary, and always changing.

If that's the case, you may be wondering if there's a way to permanently mark historical points in your project's history. For things like major releases and big merges, is there any way to mark these commits with something more permanent than a branch?
Git tags are a way to mark specific commits in your project's history with a human-readable name. Tags are often used to mark major releases, important milestones, or significant changes in the project. Unlike branches, tags are immutable and do not change as new commits are added to the repository.

Git Describe is a command that generates a human-readable name for a specific commit based on the nearest tag in the commit history. This is useful for identifying commits in a more meaningful way, especially when working with a large number of commits.


git rebase branch on top of main.

Tilde and caret operators can be chained together to specify a commit that is a certain number of commits before the parent commit. For example, git checkout main~2^ will take you to the parent commit of the commit that is two commits before the most recent commit on the main branch.



When remote repositories branch (usually origin/main) has diverged (ahead or behind) from the local branch, you can use git pull to fetch the changes from the remote repository and merge them into your local branch. This is a common way to keep your local branch up to date with the remote repository and resolve any conflicts that may arise from divergent branches. Howvwer, to keep a clean linear history, you can use git pull --rebase to fetch the changes from the remote repository and rebase your local commits on top of the fetched changes. This allows you to maintain a cleaner commit history by avoiding unnecessary merge commits.

In most cases, the origin/main is reserved for stables, reviewed code and so the origins/main is locked for direct pushes, and you have to create a branch, push to that branch and then create a pull request to merge your changes into origin/main. This is a common workflow for collaborative development, as it allows for code review and ensures that only reviewed and approved changes are merged into the main branch.

