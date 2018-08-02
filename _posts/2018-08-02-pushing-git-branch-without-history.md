---
layout: post
title: "Pushing a git branch without its history"
date: 2018-08-02
---

I recently shared a project on GitHub and wanted to keep its entire history locally without sharing it publicly. A quick search on [StackOverflow](https://stackoverflow.com/questions/12543055/#12543340) revealed the `git checkout --orphan <new_branch>` command.

Using this, the following commands allow you to save the history of a branch locally whilst uploading a fresh branch with a single commit.

Assuming you're on the `master` branch, first let's rename this branch to `old`:

``` shell
git branch -m old
```

Now we'll create a new orphan branch called `master`.

```shell
git checkout --orphan master
```

When we commit to this branch …

``` shell
git commit
```

… our commit will be disconnected from the history of the `old` branch because it won't have a parent commit. This means we can push the new `master` branch without sharing the history contained in the `old` branch.

(If you haven't yet added a remote using `git remote add origin …`, you'll need to do that first.)

``` shell
git push -u origin master
```

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
