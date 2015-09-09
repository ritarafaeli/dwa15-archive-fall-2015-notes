Branching in Git is the process of sending code commits to different *branches* of your repository.

The moment you create a branch and commit some changes to it, you create a snapshot of your codebase at that time.

Every new commit changes that snapshot.

This means that as we switch between different branches on our local computers, your codebase will actually change based on what branch you're on. It's like switching between different snapshots.



## Basic usage
When beginning to code a new feature on a project, ideally you should do so via a branch. This will allow you to commit your changes to a working branch rather than the master branch, so your changes can be reviewed by other developers without holding up the master branch from being deployed.

There are two different starting points from which you might want to create a branch:

* You know you're about to work on a new feature, so you start off with a new branch.
* You've already made some changes to some files on the new features, and now you want to put them in a new branch.

Let's assume we're starting with #1. Imagine you're working on a new promotional feature involving prize giveaways. We'll call this feature *prizes*, and we want to create a new branch to do the work for this feature.

First, run this command to see what branches already exist on your local repository:

```bash
$ git branch
```

In my current repo I see this just the `master` branch, with the asterisk indicating that's the branch I'm currently &ldquo;in&rdquo;:

```bash
* master
```

To create a new `prizes` branch, run this command:

```bash
$ git branch prizes
```

Now let's see those branches again:

```bash
$ git branch

* master
  prizes
```

With the new `prizes` branch created, switch to it using the `checkout` command

```bash
$ git checkout prizes
Switched to branch 'prizes'

$ git branch

  master
* prizes
```

Now, any commits will be made to the `prizes` branch.

Let's make some changes. Create a new file called `prize-list.txt`:

```bash
$ touch prize-list.txt
```

If we run `git status`, we see this:

```bash
On branch prizes
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	prize-list.txt

nothing added to commit but untracked files present (use "git add" to track)
```

Let's say that's all I needed to do for that feature, and now I want to commit it to Github so the developer I'm collaborating with can see my work.

First, I need to stage my changes, then I need to commit them

```bash
$ git add prize-list.txt
$ git commit -m "Create a text file of all the possible prizes"
[prizes 56ae15d] Create a text file of all the possible prizes
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 prize-list.txt
```

Now let's push this new branch. The following command says push to "origin" (remember, that's our repo on Github) our "prizes" branch.

```bash
$ git push origin prizes

Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 459 bytes | 0 bytes/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To git@github.com:susanBuck/practice.git
 * [new branch]      prizes -> prizes
```

In Github, your should see this new branch:

<img src='http://making-the-internet.s3.amazonaws.com/vc-new-prizes-branch@2x.png' class='' style='max-width:854px; width:100%' alt='New prizes branch in Github'>

If you tell Github to switch to the `prizes` branch, and then go to **Commits** you'll see your commit we just made.
If you switch it back to the `master` branch and then go to **Commits** you'll see it's not there. This is good.

At this point, your collaborator may want to look at your work on their local environment. To do that, they would run this command:

```bash
$ git pull origin prizes
```

Now they too will have that branch on their local system and they can &ldquo;checkout&rdquo; into that branch to see your code.

You can both &ldquo;pass&rdquo; back and forth code changes this way, without ever messing with the master branch.




## Merging your branch
When a feature is done and ready to be integrated into the master branch, you need to first switch to the master branch:

```bash
$ git checkout master
```

This is key: *Make sure you're in the branch you want to merge into.*

Then run git merge followed by the name of the branch you wish to merge:

```
$ git merge prizes

Updating e7e936c..56ae15d
Fast-forward
 prize-list.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 prize-list.txt
```



## Keeping your branch updated

When working on a branch you can/should try and keep it up to date with the master branch to avoid conflicts down the road. You'll do this by merging in any changes from the master branch, from within your secondary branch:

```bash
$ git checkout prizes
$ git merge master
```

If there have been changes to the master branch, you'll now see them as unstaged files which will need to be added, committed and pushed to the `prizes` branch.




## Merge Conflicts

If the files you've been editing on your branch haven't been changed on the master branch, your merge will go smoothly. If there have been changes, though, you'll get a merge conflict.

Let's look at how you can handle merge conflicts by first creating a conflict between our `master` and `prizes` branches.

On the `master` branch, edit the empty `prize-list.txt` file, adding this line of text:

```
$50 Gift Certificate
```

Then add, commit, and push this change to the master branch:

```bash
$ git add prize-list.txt
$ git commit -m 'Added a gift certificate prize'
$ git push origin master
```

Now, switch to the `prizes` branch, and again edit `prize-list.txt`. Note it's still empty in the `prizes` branch, because the change we made above only currently exists in the `master` branch.

For your edit, this time add this line of text:

```
Gold Trophy
```

Now that we've made this change, try and merge in the master branch:

```bash
$ git merge master

Updating 2144d07..985f2d4
error: Your local changes to the following files would be overwritten by merge:
	prize-list.txt
Please, commit your changes or stash them before you can merge.
Aborting
```

Here we can see git telling us there's a **merge conflict** because the version of `prize-list.txt` in the `master` branch would overwrite what we now have in `prize-list.txt` in the prizes branch.

The message tells us we can 1) *commit your changes* or 2) *stash your changes*.

Assuming we're not yet ready to commit our changes, let's look at stashing. Stashing is a way to temporarily store working changes in a branch. Run the command `git stash` to save a snapshot of your current changes:


```bash
$ git stash
Saved working directory and index state WIP on prizes: 2144d07 Second commit from prizes
HEAD is now at 2144d07 Second commit from prizes
```

Now that your changes are stashed away, you can try to merge `master` in again:

```bash
$ git merge master
Updating 2144d07..985f2d4
Fast-forward
 prize-list.txt | 1 +
 1 file changed, 1 insertion(+)
```

To confirm this worked, run `cat` on `prize-list.txt` to see it looks like the version we had made in the `master` branch (i.e. it just lists the Gift Certificate):

```bash
 $ cat prize-list.txt
 $50 Gift Certificate
```

Now, let's bring *back* the changes we had stashed away. To bring back your most recent stash, run the command `git stash apply`:

```
$ git stash apply
Auto-merging prize-list.txt
CONFLICT (content): Merge conflict in prize-list.txt
```

Note how git detected there was a conflict with what was currently in prize-list.txt (`$50 Gift Certificate`) and what we were bringing back from the stash (`Gold Tropy`).

If you view `prize-list.txt` you can see that git "auto-merged" the conflict adding what are called "standard conflict-resolution markers", i.e. the lines that start with `<<<<<<<`, `>>>>>>>` and `=======`:

```bash
$ cat prize-list.txt
<<<<<<< Updated upstream
$50 Gift Certificate
=======
Gold Trophy
>>>>>>> Stashed changes
```

Now it's up to you to do a little work and fix this merge. Edit the `prize-list.txt` file to remove the markers, and set the file how it should be, which is this:

```bash
$50 Gift Certificate
Gold Trophy
```

Once you're done, add, commit and push to your `prizes` branch:

```bash
$ git add prize-list.txt
$ git commit -m 'Resolved merge conflict, adding both gift certificate and tropy'
[prizes 2f0e492] Resolved merge conflict, adding both gift certificate and tropy
1 file changed, 2 insertions(+), 1 deletion(-)

$ git push origin prizes
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 573 bytes | 0 bytes/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To git@github.com:susanBuck/practice.git
   2144d07..2f0e492  prizes -> prizes
```

And finally, switch back to `master` and merge in prizes

```bash
$ get checkout master
$ git merge prizes
```

The end result of `prize-list.txt` should be this:

```
$50 Gift Certificate
Gold Trophy
```



## Deleting branches
If you're done with your branch, its good to clean things up (you can always branch again if you need to re-work your feature). The following will delete your branch on your local environment:

```bash
$ git branc­h -d prizes
```

And then this will remove it from Github:

```bash
git push origin --delete prizes
```
