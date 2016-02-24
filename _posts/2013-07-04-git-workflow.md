---
layout: post
title:  "A git workflow for a small group of developers"
date:   2013-07-04 11:52
categories: 2013 tools 
tags: git
permalink: git-workflow
published: false
---
# A git workflow for a small group of developers
In a fast pace project which involves several programmers, the number of commits in the main repo skyrockets in a short period of time. Keeping this flow under control is important to ease tracking changes and to effortlessly spot the bad commit which introduced the latest bug.

> *Many hands make light work. Lots of commits, though.*
> *(Unknown Jedi Master)*

There are lots of articles describing git practices and tools to support them, like [git-flow](https://github.com/nvie/gitflow), [git_remote_branch](https://github.com/webmat/git_remote_branch)... After reading and trying almost all of them, I realized they were fine but a bit bloated when your project is smaller, your app isn't live yet and/or your workflow differs from theirs, so I stick to some simple rules which suit the dynamic nature of my projects:

* **craft descriptive commit messages**: describe the implemented feature or the fixed bug. I usually precede the message with the id (i.e. a ticket number) of the task which motivated the code, with the string BUGFIX if the change is a fix, or TYPO when fixing those stupid typograhic errors.
Depending on the collaboration platform you use, it's possible that, by prefixing with a special code, you get bonus automatic features like linking the commit as a comment to an specified ticket, firing specific builds on your CI server and so on.
* **avoid unstable commits** (not well tested and/or unfinished code) **in the main branches of the project** (usually master and develop): Errare humanum est, specially when you aren't doing TDD/BDD. Working on local repo, I sometimes realize there is a bug in my code the moment after committing. Fixing it and amending that last commit (*git commit --amend -C HEAD*) is quite handy to avoid creating a new commit to fix it and leave the previous unstable one in our repo history.
If someone pulls from main develop branch, they will get unstable code. If someone wants to use *git bisect* to detect the first commit where a bug was introduced, they will be stuck on that unstable commit without any means of testing the app and determinining if it's a good or bad commit.
* **avoid artificial commits**: if you usally do git merge, or follow some established git flow (like nvie) and/or use some tools (like grb, git-flow...) which work following those processes, your merges always are non fast-forward unless there is only one commit in your branch, so you get log entries like:
    Merge branch 'my_feature' into develop
    Merge branch 'develop' of git.example.com:my_project into develop.

First one is result of a non fast-forward merge. The other, the result of updating our branch merging from develop or pulling from origin after merging our branch but before pushing our commits.

In the repo I studied to research this article, at the time of this writing, there were 3019 commits. A total of 745 (24.67)% commits like 'Merge branch...'. 461 (15.26%) out of those merge commits were of the kind 'Merge branch develop into…'.
## How/Why can we avoid those spurious commits?
Let's see two different flows with two programmers, A and B:
### Non fast-forward merge

```
    A initial import
    B clone
    A git-flow init
    A git-flow feature start a
    A commits
    A commits
    A git-flow feature finish a
    A push origin develop
    B commits
    B commits
    B git-flow feature finish b
    B git pull origin develop
    B git push origin develop
```

The log:

```
    d99f505 Merge branch 'develop' of ssh://bitbucket.org/ejosafat/git_flow_example into develop
    9339606 Merge branch 'feature/b' into develop
    26be7bf B commit 2
    09bb8ee B commit 1
    fecb8f0 Merge branch 'feature/a' into develop
    4839a47 A commit 2
    8454612 A commit 1
    54b3b7a Initial import
```

### Rebase and fast-forward merge
    A initial import
    B clone
    A git checkout -b ej_a_feature
    B git checkout -b ej_b_feature
    A commits
    A commits
    A git pull origin develop
    A git push origin ej_a_feature:develop
    A git checkout develop
    A git branch -D ej_a_feature
    B commits
    B commits
    B git pull --rebase origin develop
		B git push origin ej_b_feature: develop
		B git checkout develop
		B git branch -D ej_b_feature
Log:
    0dc2820 B commit 2
    4ead119 B commit 1
    08af32b A commit 2
    f4b9db6 A commit 1
    6001a1f Initial import

As you can see, second history is clearer than first one.

Some people are against this approach because they prefer to keep the track of the original branch those commits belonged to, so they show as a group of changes to implement a specified feature, and craft the message of the non fast-forward commit to reflect the name of the feature and so on.
My personal experience is that in a typical project, no one bothers to craft descriptive messages for the empty commit which groups them all, and nobody cares about seeing them as a group.
For further discussions about this, you can read these interesting articles:

* [article against rebase](http://paul.stadig.name/2010/12/thou-shalt-not-lie-git-rebase-ammend.html)
* [article favouring rebase](http://lwn.net/Articles/328438/)

## To keep in mind
* **Don't ever rebase your branch if some of its commits have been already merged into the develop branch**. I'm not going to explain in detail the inner-working of rebase, you only need to know that you'll get duplicate commits applied to your develop branch after you finally merge your code.
* If you do code reviews as part of your workflow (and you should at least for the critical features), then you must use non fast-forward commits because you need to examine all the code related to an specific pull request.
* Rebase at least once a day agains the current develop branch, this avoids lots of merge conflicts. In face paced development environments which evolving API's, it ensures you code still works when you finish the feature and even more importantly, you don't break anyone's code. If you have uncommitted changes before rebasing, use *git stash*.
* It sometimes occurs that someone pushes just after you rebased and before your push. Simply rebase again and try to push again.
* Sometimes you make a WIP commit so you can push your branch to the remote server and fetch it from another machine. That commit is unstable by definition, so you can use the interactive option of git rebase and rewrite your commit history to get rid of it.
* It's safe to push your code to your own remote branch as a backup measure, but remember to delete it after fetching it in another computer and always after a rebase against the main develop branch. You can always push a new one after your day's work. This is a good safety measure to avoid commits history nightmares.
* Do a favour to your mates: run the full test suite before pushing to main develop branch. Don't wait for the CI server to warn you.
* For those who prefers to use a GUI for git, this flow is perfectly doable using SourceTree.
