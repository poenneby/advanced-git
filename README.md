# Advanced Git

## Prerequisites

### All you need is Git

This hands-on assumes you have a __Git__ client installed.

MacOS and Linux users will already have Git installed.

For Windows users we recommend installing __Git Bash__ which emulates a Linux shell with bash and Git installed.

You will also need a couple of handy aliases to visualise the history of the application.

```
git config --global alias.lol "log --graph --decorate --pretty=oneline --abbrev-commit"
git config --global alias.lola "log --graph --decorate --pretty=oneline --abbrev-commit --all"
```

## Keep the history tidy

We will continue working on a version of the application that was developed throughout __Practical Git__

Start off by cloning the random-number-generator repository.

```
git clone https://bitbucket.mycloud.intranatixis.com/scm/fk7/random-number-generator.git
```

Let's look at the history of this project
```
git lola
```

```
* e283459 (HEAD -> master, origin/master, origin/HEAD) Confirm a file has been generated
| * 773df79 (origin/feature-4) Factor out duplication
| * a1a1c13 Inform user of file written
| * 8cc6caf (origin/feature-3) Fix indentation
| * 74f38f1 Fix typo
| * 98b56ee Show usage when filename omitted
|/
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

There are some active branches with finished work and our goal is to merge all the work into master

One approach we could take is to try to merge branch by branch straight into master

Let's try merging the branch `feature-3`

```
git merge origin/feature-3
```

It merged ok using a recursive strategy - but we got ourself a merge commit which does not make much sense or add value

```
*   d99e7c7 (HEAD -> master) Merge remote-tracking branch 'origin/feature-3'
|\
| * 8cc6caf (origin/feature-3) Fix indentation
| * 74f38f1 Fix typo
| * 98b56ee Show usage when filename omitted
* | e283459 (origin/master, origin/HEAD) Confirm a file has been generated
|/
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

Let's continue  merging feature-4

```
git merge origin/feature-4
```

We get a conflict in the `generator.sh` which had been modified in a commit ahead in master

We solve the conflict by keeping the work in the feature branch and add another commit to mark the conflict as resolved

```
git checkout --theirs generator.sh
git commit -am "Resolved conflict"
```
(Super useful commit message, I know...)

Again have a look at the history

```
git lol
```

```
*   65b72e7 (HEAD -> master) Resolved conflict
|\
| * 773df79 (origin/feature-4) Factor out duplication
| * a1a1c13 Inform user of file written
* |   d99e7c7 Merge remote-tracking branch 'origin/feature-3'
|\ \
| |/
| * 8cc6caf (origin/feature-3) Fix indentation
| * 74f38f1 Fix typo
| * 98b56ee Show usage when filename omitted
* | e283459 (origin/master, origin/HEAD) Confirm a file has been generated
|/
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

That's getting messy. Imagine trying to track down a change in that jungle of commits? 

Is there was a tidier way of doing this?

### Rewind

Let's start by removing the last 2 merge commits

```
git reset --hard HEAD~2
```

Let's focus a little bit a on the quality of our git history

It's always a better idea to `rebase` (update) our feature branches instead of the master branch

We will start by tidying up the individual branches

Checkout our first feature branch and rebase it interactively:

```
git checkout feature-3
git rebase -i HEAD~3
```

Your configured EDITOR will open the following file showing the last 3 commits

```
pick bde82cc Show usage when filename omitted
pick e3f3c77 Fix typo
pick 9b25aa3 Fix indentation

# Rebase 9721085..9b25aa3 onto 9721085 (4 command(s))
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
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out

```

We only want to keep the one commit `Show usage when filename omitted` so we use `fixup` or `f` to squash the commit into its' previous commit and discard the log message

```
pick bde82cc Show usage when filename omitted
f e3f3c77 Fix typo
f 9b25aa3 Fix indentation
```

Git will rewrite your log when you save and close this file and you should be left with a single commit in your branch

```
> git lol

* 0a2246a (HEAD -> feature-3) Show usage when filename omitted
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

We observe that the `feature-4` branch has been created from `feature-3` thus it will include some of the commits we just squashed

Let's update this branch by rebasing it with `feature-3`

```
git checkout feature-4
git rebase feature-3
```

You will get conflicts for each previously squashed commit but you can safely skip them like so

```
git rebase --skip
git rebase --skip
```

While we are still in branch `feature-4` we rebase it interactively to again squash the refactoring commit

```
git rebase -i HEAD~2
```

Again we fixup the last commit leaving us with one cohesive commit.

```
pick b87db6c Inform user of file written
f 03f5298 Factor out duplication

# Rebase 3ff5c43..03f5298 onto 3ff5c43 (2 commands)
#

...
```
Your log (as seen from feature-4) should now look similar to this:

```
git lol

* ba63cd0 (HEAD -> feature-4) Inform user of file written
* 0a2246a (feature-3) Show usage when filename omitted
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

Now that the individual feature branches are clean we rebase them with the `master` branch

```
git checkout feature-3
git rebase master
```

The rebase rewinds HEAD to replay the branch commits after the commits in master

We are done preparing our branch and can merge now merge and delete it

```
git checkout master
git merge feature-3
git branch -D feature-3
```

We do the exact same for `feature-4`

```
git checkout feature-4
git rebase master
```

We resolve the conflict by using the feature branch code

```
git checkout --theirs generator.sh
git add .
git rebase --continue
```

With the branch up to date with `master` we go ahead and merge

```
git checkout master
git merge feature-4
git branch -D feature-4
```

If you followed along the steps your log should look similar to this

```
git lol

* 16fe226 (HEAD -> master) Inform user of file written
* 89f42f9 Show usage when filename omitted
* e283459 (origin/master, origin/HEAD) Confirm a file has been generated
* 9721085 Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

That's better!


## Help! I've lost all my work 

Imagine you have come out of a particularly difficult merge of a long lived outdated branch involving several conflict resolutions.
And now you realise some of the commits that you had before are no longer there.

Don't worry, Git remembers...

Let's try this by deleting the last 2 commits in our current repository

```
git reset --hard HEAD~2
```

Using `reflog` we can list all the commands that's been run including the related SHA1

```
git reflog

e283459 HEAD@{0}: reset: moving to HEAD~2
16fe226 HEAD@{1}: merge feature-4: Fast-forward
89f42f9 HEAD@{2}: checkout: moving from feature-4 to master
16fe226 HEAD@{3}: rebase finished: returning to refs/heads/feature-4
16fe226 HEAD@{4}: rebase: Inform user of file written
89f42f9 HEAD@{5}: rebase: checkout master
ba63cd0 HEAD@{6}: checkout: moving from master to feature-4
89f42f9 HEAD@{7}: merge feature-3: Fast-forward
...
```

In our case we identify the SHA1 that correspond with the merge of feature-3 and feature-4 and `cherry-pick` these

For example:
```
> git cherry-pick 89f42f9
> git cherry-pick 16fe226
```

And our history has been restored 



### Find the buggy commit using bisect

#### TODO





