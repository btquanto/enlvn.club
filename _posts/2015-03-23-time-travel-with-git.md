---
title: Time travel with git
is_recommended: true
category: Git
tags: ["git", "tutorial"]
---

# Scenario:

A thing about using `git rebase` is that it changes git history, and changing history is always risky (you're not a [Time Lord](http://tardis.wikia.com/wiki/Time_Lord) or a [time agent](http://tardis.wikia.com/wiki/Time_Agent))

Occasionally, you mess up. Running `git log` gives you a messed up git history, with duplicated and/or missing commits, and multiple merges of the same branch, and blah... blah...

On the bright side, you are using git, and that gives you the power of a [time traveler](http://tardis.wikia.com/wiki/Time_travel).

# Saviors of the day: 'git reflog' and 'git reset'

A good thing about git is that all your committed changes are never truly gone (This is not actually true, though: traceable commits (commits whose children link to a t least one reference) are always ...traceable; but orphaned commits will be gone in around 2 weeks by default, and thus, you have around 2 weeks to save your lost work), and can be traced back using `git reflog`.

This is an example of what produced by `git reflog`.

```
07f84b9 HEAD@{0}: checkout: moving from master to demo
41b8897 HEAD@{1}: reset: moving to origin/master
a54fdd2 HEAD@{2}: checkout: moving from demo to master
07f84b9 HEAD@{3}: merge master: Merge made by the 'recursive' strategy.
11b3cff HEAD@{4}: reset: moving to origin/demo
a54fdd2 HEAD@{5}: merge master: Fast-forward
11b3cff HEAD@{6}: checkout: moving from master to demo
a54fdd2 HEAD@{7}: rebase -i (finish): returning to refs/heads/master
a54fdd2 HEAD@{8}: rebase -i (pick): Hotfix: Check for AssetsConfirmed required value
c2a2f45 HEAD@{9}: rebase -i (pick): Move checkEmptyField into a separate javascript utils file
11b3cff HEAD@{10}: rebase -i (start): checkout demo
41b8897 HEAD@{11}: rebase: aborting
11b3cff HEAD@{12}: rebase: checkout demo
41b8897 HEAD@{13}: checkout: moving from demo to master
```
In the above example, you can see all the actions you have been doing to your git tree, and all the references to all the `HEAD`'s that you have been on.

For example:

```
41b8897 HEAD@{1}: reset: moving to origin/master
```

means that you have called

```
git reset origin/master
```

which makes the current `working directory` points to `41b8897`. The hash `41b8897` is what would be used for a `git reset` (in case you want to reset your current branch to that point of time)

```
`git reset` changes the current branch's address to another git commit.
```

For example, I am currently on `demo`, and want to point `demo` from wherever to the current `origin/master`, I can do either

```
git reset origin/master --hard
```

or

```
git reset 41b8897 --hard
```

_Note: specifying `--hard` forces resetting the content of the current directory to the destination commit, not only the reference of the current branch_

**Official documentation**:

* [Git-reflog](http://git-scm.com/docs/git-reflog)
* [Git-reset](http://git-scm.com/docs/git-reset)
