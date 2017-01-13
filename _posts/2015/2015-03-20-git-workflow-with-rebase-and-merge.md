---
layout: post
title: Git workflow with rebase and merge
is_recommended: true
category: Git
tags: ["git", "tutorial"]
---

**Example branches**

* Remote: `origin`
* Feature branch: `feature_branch`
* Destination branch: `demo`

---

**Detailed step by step (meaning, this is almost certainly fool-proof, but the steps can be reduced):**

1. **Enable `rerere` (Only once in every development machine)**
    ```
    git config --global rerere.enabled true
    ```
    _Explain: `rerere` allows conflicts resolves between commits be saved, so when the same conflicts happens again, you don't have to resolve it again. This is helpful in processes that do a lot of `git rebase`_
2. **Update feature branch for any newly pushed changes**
    ```
    git fetch
    git rebase origin/feature_branch
    ```
    _Note: When rebasing, conflicts may happens, depending on how far off from your `feature_branch` to the `demo` branch. It's a good practice to reset the demo branch after every deployment cycle_
3. **Push feature branch to origin (For a remote backup)**
    ```
    git push origin feature_branch
    ```
4. **Update destination branch (No need to fetch, since we have just already fetched once)**
    ```
    git checkout demo
    git rebase origin/demo
    ```
5. **Rebase the current branch with the destination branch**
    ```
    git checkout feature_branch
    git rebase -i demo
    ```
    _Note: Specifying `-i` allowing choosing only the commits that have not already been in the destination branch, thus reduce chance of conflicts, and avoid duplicated commits_
6. **Merge feature branch  to destination branch**
    ```
    git checkout demo
    git merge --no-ff feature_branch
    ```
    _Note: Specifying `--no-ff` forces creating a merge commit, and thus groups related commits together, leaving a clean and traceable git history_
    ![Compare non-fast-forwarded with fast-forwarded merging](/public/images/merge-without-ff.png)
7. **Push the destination branch to remote**
    ```
    git push origin demo
    ```
8. **Reset the feature branch to the backup that has been pushed to demo**
    ```
    git checkout feature_branch
    git reset origin/feature_branch --hard
    ```
