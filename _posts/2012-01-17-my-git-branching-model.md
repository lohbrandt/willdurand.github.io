---
layout: post
title: My Git branching model
location: Clermont-Fd Area, France
credits: |
    Vincent Driessen for his [Git model](http://nvie.com/posts/a-successful-git-branching-model/)
    and illustrations.
updates:
  - date: 2022-03-12
    content: I proofread this article and updated the figures.
---

We all probably know [this successful Git branching
model][successful-git-model], which looks like a very interesting model for
teams that want to use Git. That being said, this model is a bit too complex for
common needs in my opinion. In this article, I introduce my lightweight model.

I use two main branches:

- `main` : the code in a _production-ready_ state;
- `develop` : the _integration branch_.

![](/images/posts/2012/01/git-develop-main.webp)
_Figure 1: Commits over time on two branches: "develop" and "main" (based on
Vincent Driessen's similar illustration)_
{:.with-caption .can-invert-image-in-dark-mode}

I also use **feature branches**. A feature branch contains a work in progress.
Keep in mind that a feature branch should reflect a feature in your backlog. I
use a convention for these branches, I always prefix them with `feat-`.

    $ git branch
    feat-my-feature
    * main

A feature branch has two constraints:

- the code must be based on the `develop` branch;
- the code must be merged in the `develop` branch.

![](/images/posts/2012/01/git-feature-branch.webp)
_Figure 2: A feature branch next based on the "develop" branch (based on Vincent
Driessen's similar illustration)_
{:.with-caption .can-invert-image-in-dark-mode}

To create a feature branch, I use the following command:

    $ git checkout -b feat-my-feature develop

To merge a feature branch into `develop`, I use the following set of commands:

    # Go back to the develop branch
    $ git checkout develop

    # Get last commits
    $ git pull --ff-only origin develop

    # Switch to the feature branch
    $ git checkout feat-my-feature

    # Time to rebase
    $ git rebase develop

    # Then, switch to the develop branch in order to merge the feature branch
    $ git checkout develop

    $ git merge --no-ff feat-my-feature

    # Push
    $ git push origin develop

    # Finally, delete your branch
    $ git branch -d feat-my-feature

I always merge a feature branch into `develop` using `--no-ff` to keep a clean
log:

![](/images/posts/2012/01/git-merge.webp)
_Figure 3: The difference between `git merge` and `git merge --no-ff` (based on
Vincent Driessen's similar illustration)_
{:.with-caption .can-invert-image-in-dark-mode}

The `--no--ff` option allows to keep track of a feature branch name which is
quite useful. The following `git log` output shows you a feature branch merged
with this option:

    commit 481771556824c4ae2e6da73ef14d6ce757fb5870
    Merge: 6abdd70 8cfe5a7
    Author: William Durand <email address>
    Date:   Tue Jan 17 11:31:56 2012 +0100

    Merge branch 'feat-my-feature' into develop

    commit 8cfe5a7da159663cc09a850bee49a59ce046c67e
    Author: William Durand <email address>
    Date:   Tue Jan 17 11:31:19 2012 +0100

    Added a new feature

    commit 6abdd707aace50ee5aad72a3c6fcff2f36cdea7f
    Author: William Durand <email address>
    Date:   Sun May 15 14:07:19 2011 +0200

    Initial commit

Without the `--no-ff` option, you'll get the following output:

    commit 0d5805d52e55e4941ce23585a4cd559e5e643207
    Author: William Durand <email address>
    Date:   Tue Jan 17 11:35:43 2012 +0100

    Added yet another feature

    commit 6abdd707aace50ee5aad72a3c6fcff2f36cdea7f
    Author: William Durand <email address>
    Date:   Sun May 15 14:07:19 2011 +0200

    Initial commit

In a team, you will probably have more than one feature branch, and you could
have a dependency between two branches (this should be avoided). In this case,
I use another branch in which I merge two or more feature branches.

    $ git checkout -b feat-my-feature-with-another-feature develop

Then, I can merge the two feature branches, and solve possible conflicts:

    $ git merge feat-my-feature

    $ git merge feat-another-feature

I don't use any other branches. The last part of the model is to merge `develop`
into `main`. To avoid conflicts, there should be only one person who owns
this responsibility: the **release manager**.

I experimented this model with different teams in terms of number of people and
skills, and I never had more needs. I know some people use **release** branches
but it can be handled in another way.

[successful-git-model]: http://nvie.com/posts/a-successful-git-branching-model/
