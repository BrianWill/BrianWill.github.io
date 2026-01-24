# Introduction to Jujutsu Version Control

Jujutsu, otherwise known as jj, is a version control system that has a number of notable advantages over git:

1. the first of which is that it allows you to effectively mutate commit history even though, like in git, the commits are strictly immutable. Among other things, mutable history makes certain workflows much easier, in particular stacked diffs. If you’ve ever had to split a large PR into multiple smaller PRs, you’ve probably experienced pain when you want to propagate changes between the PRs and keep them in sync. While some external tools can help you manage this pain in Git, JJ makes it easy out of the box.
1. A second advantage jj has over git is that it stores conflicts as commit meta-data, which as we’ll see later, can make conflicts easier to resolve when merging.
1. And third, unlike git, jj has no concept of an index, so commits do not have to be staged. Instead, running any jj command will commit any dirty changes in your working copy before doing anything else. Effectively, then, you never need to stash, and you never get stuck in the middle of a merge. You can always just switch away to any branch at any time without losing any work.

Now if there are drawbacks to jujutsu:

1. the first is that it hasn’t yet reached version 1.0
1.and though it is generally reliable at this point, there has been some instability in the command interface.
1. The other notable drawback is that jj doesn’t currently support git-lfs, so it may not be a practical choice for projects with numerous or large binary files.

jj is architected to support swappable storage backends, but the only backend fully supported at the moment is git itself. When using this backend, a .jj repo directory and a .git repo directory sit side-by-side in the root of your working copy, and every jj commit is stored as a git commit, along with some additional jj meta-data. Likewise, the full command history of your jj repo is stored as additional git commits.

When cloning from or synching through a remote repo, the remote is just a regular git repo with no knowledge of jj. The git commits contain all the jj meta-data, so jj users can sync just with push and fetch. In fact, you can clone any git repo and work with it on your own as a jj repo, even if other users of the git repo do not.

Let’s now look at the most basic operations. First we’ll create a new project directory, and then we’ll use the jj git subcommand to init the dot jj and dot git repo subdirectories. Once inited, we can now view the repo history with jj log.

Note that jj’s git subcommand has several of its own subcommands for working with git remotes and the underlying git repo, but despite some of these subcommands sharing the same name as git subcommands, such as init, clone, fetch, and push, you should think of these jj git subcommands as distinct from actual git’s own subcommands. On the other hand, the underlying dot git repo can be operated upon directly with git just like any normal git repo. Just keep in mind you may need to sync changes made directly through git back to the jj repo with the command jj git import. Normally though, you’ll manage a jj repo through jj itself rather than use git directly.

Anyway, to create a commit, we simply make changes to our working copy, and then run jj. Any time jj runs, it will make a new commit for any dirty changes in the working copy directory. This is true for any jj subcommand, even just log: jj will still commit any dirty changes.

Unlike git, jj does not have a concept of a current, checked out branch: instead, there is simply a current commit, and when we want to switch our working copy to another commit, we specify a commit id with the edit subcommand. So here, assuming there is a commit id that uniquely starts with 5d11, we can switch our working copy to this commit with the command jj edit 5d11

Again, understand that any jj command will make a commit if the working copy has dirty changes. So if we create a new file and then switch again to a different commit with jj edit, jj will first make a new commit before switching. Effectively, no matter the state of your repo and working copy, you can always switch at any time. If you don’t like any changes that get committed, don’t worry: as we’ll see later, you can easily remove them from your visible commit history.

Before demonstrating more commands, we have to describe the Jujutsu data model and how it differs from Git. Namely, jj has hidden commits, an operations log, change ids for each commit (in addition to the commit ids), bookmarks instead of branches, conflict metadata that’s stored with each commit, and the possibility for individual commits to have more than two parents.

First, hidden commits: every jj commit has a visibility flag, and various commands will toggle this flag. What it means to be hidden is simply that hidden commits will be omitted by default when displaying history with commands like jj log. This is helpful for users because it removes clutter from the history, particularly when old commits get logically replaced by newer versions.

The operations log is an immutable history that tracks every command which modifies the state of the repo. With each command, the log records the set of commits that were visible after the command executed, and this allows us to restore the repo back to any prior state. Be clear, then, the commands which restore an old state are themselves appended to the log and never actually create or delete any commits: instead, an old state is restored simply by toggling which commits are visible.

In addition to the normal git content hash commit ids, jj commits also have a change id. These change ids are represented with only lowercase English letters, and they are either randomly generated or inherited from a prior commit, depending upon the operation that creates them. Most of the time, a repo will have one visible commit at a time for an individual change id, but there are scenarios where a repo may have multiple visible commits simultaneously for an individual change id. This situation is called a “divergent change”. While divergent changes are not error states, per se, they do make your commit history a bit confusing, so normally you’ll rectify the situation in one of a few ways, such as by hiding all but one of the commits or maybe by merging the divergent changes together.

As mentioned, jj can store conflict meta-data in commits. For example, if a commit has two parents with a conflict in a certain file, the irreconcilable difference is stored in the commit alongside each parent’s version of the file. Commits with these conflicts are marked in the log. Like divergent changes, conflict commits in your repo are not an error state, per se, but normally you’ll want to rectify the situation by making new child commits that resolve the conflicts, or alternatively, you might simply hide a commit with conflicts, which may be appropriate if you just want to abandon a merge.


Instead of branches, jj has what it calls “bookmarks”, though they are basically the same thing. The main difference is that jj has no concept of a “current bookmark”, and bookmarks do not automatically advance the way git branches do. As we’ll see, it’s common in jj to locally track different branches of work by change ids rather than with bookmarks, so bookmarks are mostly used in jj just to sync with remote branches.

If a bookmark that tracks a remote branch somehow ends up out of sync with the remote branch, say because the bookmark was manually moved, then the bookmark becomes conflicted. These conflicts can be resolved by simply getting the bookmark and its tracked remote back in sync.



Lastly, whereas a git commit can have at most two parents, a jj commit can have any number of parents. This is mainly useful for cases where you want to do a multi-way merge: instead of having to merge multiple pairs, you can just merge everything together directly in one operation. Along with jj’s conflict meta-data, this can help reduce the number of conflicts you must manually resolve.


Before getting back to demonstrating more commands, we’ll note that jj’s use of the term “change” is unfortunately confusing. For starters, when we talk about a “change” in jj, it’s not always clear if we’re talking about a change id or if we’re talking about a commit that has a particular commit id. More confusingly, the term “change” can also refer to the actual file content changes of a commit relative to its parents, and the jj documentation is not always clear about this distinction. These ambiguities would have been avoided if “change ids” were instead named something else, like perhaps, “revision ids”, but as it is, we’ll have to be careful about these distinctions when we say “change”.


So now, let’s look at an actual command line and recreate our test project 

First we’ll create the project directory, cd, then create the repo with jj git init. Quick check with ls -la shows the .jj and .git subdirectories. In this initial state, if we check the output of git log, we get an error: “your current branch ‘master’ does not have any commits yet”. If we check jj log, though, it shows we have two jj commits:

The root commit of a jj repo always has commit id of all zeros, shown in blue on the right side, and a change id of all z’s, shown in purple on the left. On top of the root commit, we have one other commit, which is marked with the @ symbol to indicate that it’s the current commit. The commit id d7380b97 and the change id ‘spotollr’ was randomly generated. The commit is marked as ‘empty’ because it has no content changes relative to its parent, the root commit. 

“no description set” indicates that the change has no commit message, but we can easily fix that with the command jj describe, where the -m option specifies the message. Here we’ll add the description “my first change”

The response output indicates that the working copy has been updated with a new commit, and because the new commit shares the same change id, it shares a logical identity with the old commit. So it may look like we’ve edited the existing commit, but in truth it remains unchanged. Instead, the old commit was replaced and hidden. If we look at jj log again, we still only see the root commit and a single commit with change id ‘spotollr’. The original commit with this change id is still in our repo, though, as we can see if we run jj evolog (short for evolution log). This command shows us all commits that have a particular change id, in this case the change id of the current commit.

By the way, you may have noticed that jj displays commit and change ids with color coding that helpfully indicates which digits are sufficient to uniquely specify the id. For example, the change id ‘spotollr’ is displayed with only the first letter in color because currently no other commits in the repo have a change id that begins with ‘s’. As long as this holds true, we can just use ‘s’ by itself to specify this change id in jj commands.

You’ll also notice in this evolog output that each commit specifies the operation in which it was created with an operation id. We can see the full history of operations with the command jj operation log.

At any time, we can go back to the repo state after any prior operation with the command jj operation restore. For example, if we restore back to the state before we added the commit description, jj log now shows that change ‘spotollr’ no longer has a description. Be clear, though, that restoring an old state does not actually create or destroy any commits: instead it just toggles visibility of all the commits to match the chosen state. Also be clear that the command log itself is not unwound: instead, the restore operation itself is just appended to the end of the log.

As shorthand, we can go back to the state before the last operation with jj undo, but this is just the same as using jj operation restore to go back one operation.

At any time we can switch our working copy to any commit with the jj edit command. The one exception is the root commit is treated as immutable. [show error if we try jj edit 00000]

Looking at the evolog, though, we can go back to the earlier commit with no description. 

Now jj log shows something a bit odd: switching to the old commit has made it visible again, and so now the log indicates with double question marks that we have a divergent change: there are currently two separate visible commits having the same change id, ‘spotollr’. 

As we mentioned earlier, a divergent change is not really an error, but it is something we usually want to fix in normal workflows because we generally will rely upon change ids to uniquely identify our branches of development. Also, as long as a change id is divergent, we can’t use the change id in commands as an unambiguous reference to a single commit.

How we might fix a divergent change depends upon the scenario, but let’s say here we decided we don’t like the newer changes we made and so want to discard the newer commit. We can do that with jj abandon. (Notice that, a bit confusingly, the newer commit here is shown lower in the log, breaking the normal expectation that the log shows commits in reverse chronological order.)

In this case, the commit we’re abandoning is a leaf commit without any children of its own, so jj abandon will just hide the commit. If the commit did have descendents, though, jj would actually replace all of its descendents with new commits that effectively remove all the changes inherited from the abandoned commit.

To see this in action though, we’ll first need to create commits with new change ids. In a normal git workflow, a sequence of incremental commits creates a chain of descendents, but when we “edit” a change in jj, each incremental commit logically replaces the prior version and so has all the same parents. So what the evolog shows is actually misleading: despite what the line on the left margin implies, these commits are not actually related by parentage; instead, they just share the same change id and are being listed in reverse chronological order. 

Anyway, let’s finally create some commits with distinct change ids, which we can do with the command jj new.

Without any arguments, jj new switches the working copy to a new empty commit that is a child of the prior, and this new commit has a new, randomly generated change id. So looking at the log, we see our new current commit has change id xmuqxnmy, and this commit has change ‘spotollr’ as its parent.

Let’s now create a file foo and make a new commit. In the log, you see the current commit then has a new commit id but shares the same change id, xmuqxnmy.

Let’s run jj new again to create another new child with a new change id, this time ‘suwrqwut’. 
We’ll create another new file bar and commit. 

Our project directory now contains two files, foo and bar, but let’s see what happens when we edit the change in the middle, xmuqxnmy. 

First let’s just set the description with jj describe. This is the change that added file foo, so let’s say jj describe -m “added file foo”. As expected, change xmuqxnmy now been replaced with a new commit, but less expected, the child suwrqwut has also been replaced with a new commit. This is necessary because, like in git, parent references are immutably baked into each commit and so a new commit must be made for suwrqwut that references its new parent.

Next let’s edit the change’s content. In our working copy, we already have the file foo, but it is empty. Let’s add some content to the file and commit. Looking at the log now, like before, both change xmuqxnmy and its child suwrqwut have new commit ids, but if we switch back to the child, we can see that the file foo now has the content we just added to the parent. What this illustrates is that jj treats commit history like a stack of changes rather than just a series of snapshots, and so any edits to a change will propagate to its descendents.

Finally then, let’s see what happens when we abandon the parent change, xmuqxnmy. Not only is the commit hidden, the content of the change is effectively removed from its descendents, as if we rewound and replayed the full history but skipped the abandoned change. So now, because file foo was added by the change we just abandoned, change suwrqwut no longer contains file foo.

Let’s now talk about conflicts. Conflicts can, of course, arise when merging, but they can also arise in scenarios where we edit changes with ancestors. To demonstrate, let’s continue where we left off and again run jj new to create a child commit with a new change id, zltnmzwr. In this new change, let’s add a line of text to the empty file bar. Then let’s switch back to the parent, ‘suwrqwut’, where the file bar is still empty, but now we’ll add a different line of text to the same file. When we then run jj log, we can see that the child, ‘zltnmzwr’, is now marked as having a conflict. This happened because the line was modified in the parent change after it was already modified in its child. While it’s normal for descendents to clobber the changes of their ancestors, the other way around may be unexpected, so jj marks such cases as conflicts.

So if we now switch back to the child, zltnmzwr, and look at the bar file, the conflict is denoted with angle brackets similar to the standard syntax used in git. 

Like with divergent changes, commits with conflicts don’t really break our repo in any way, so we could just ignore or maybe abandon the conflicting commit, but assuming we want to resolve the conflict, we can simply edit the file to remove the conflicts and then run jj to commit the fix.

Alternatively, we could use the jj resolve command to interactively navigate and resolve the conflicts.

Now let’s finally demonstrate a merge. First, we’ll give change zltnmzwr a sibling by running jj new suwrqwut. This creates a new child of suwrqwut with change id rysnwkyn. Before merging, let’s modify the single line of the file bar so as to create a conflict when we merge. [echo ‘lorem ipsum’ > bar]

Now to actually perform the merge, we simply run jj new with both zltnmzwr and rysnwkyn as arguments: 

jj new zltnmzwr rysnwkyn

This creates a commit with two parents, and because the new commit doesn’t introduce any new changes relative to its parents, it is marked empty. However, because the changes inherited from the two parents modify the same line of file bar in different ways, the new commit has a conflict.

As with any other conflict commit, we can resolve it by manually editing the conflicting files or using the interactive jj resolve command. [show resolving manually]

REMOTE GIT REPO

Now let’s demonstrate how to use jj with a remote git repo.

Instead of continuing with our test repo, let’s create a new github repo, which we’ll call jj-remote. Before cloning, we’ll create a file foo with a few lines directly on github in the main branch. Then in the directory where we want to put the local repo, we start with: 

jj git clone https://github.com/BrianWill/jj-remote.git

Then we cd into the new jj-remote directory, where we can run ls -la to see the dot jj and dot git subdirectories.

Running jj log shows two changes, one of which is referenced by the local bookmark main, which tracks the branch main in the remote repo. The other change, its child, is our current working commit. The clone creates this new change with the idea that we should generally start our new work in a fresh empty change.

So let’s make a simple change that we want to push, let’s say creating a new file bar. For good measure, we’ll edit the description to indicate what this change does:

jj describe -m “new file bar”

Now assuming we intend to submit PRs on separate branches rather than pushing directly to main, the way to proceed is to create a new local bookmark that will track a new remote branch and then push this tracked bookmark. We can do this all in one command:

jj git push --change @

The change flag creates a new local bookmark and remote branch that will be automatically named “push-” followed by the changeid of the specified commit, which here is the symbol @ , shorthand for the current working commit. So now if we run jj bookmark list to display the local bookmark, we’ll see two: the bookmark main that was created upon clone, and then also the bookmark push-vvunluysoppw that was just created for our push.

On github, we can create a pull request from the new branch. [demo this] 

Let’s now edit our local change by adding another file:

touch ack
jj log

To then push this change to the remote branch, we can run jj git push with no args, which pushes all of the tracked bookmarks. To push just a single bookmark, we can use the --bookmark option to specify the bookmark by name. Annoyingly, though, bookmark names cannot be specified by shorthand, so you’ll have to type out the full name.

Now if we look again at our PR on github, it has the new added file, but confusingly it shows only one commit rather than two. What happened here is that our change was revised with a new commit that replaced the prior, and so then the remote branch was updated to this new, unrelated commit. The old commit is still in the remote repo, but it is now orphaned without any branch pointing to it, so it should be garbage collected at some point in the future.

Now let’s see what can happen if someone else coclear with us on this same PR. First we’ll make a separate clone of the same repo. This separate clone doesn’t initially have a bookmark for our PR branch, so we’ll set one up manually with the command:

jj bookmark track push-vvunluysoppw@origin

This creates a local bookmark that tracks the remote branch named push-vvunluysoppwm. Now we’ll switch to the new bookmark’s commit and modify the file ack:

jj edit push-vvunluysoppw # we can specify the commit by its bookmark
echo ‘repo2 change’ > ack

Then we’ll push the new modification:

jj git push

Looking at the PR again on github, again the PR still shows just one commit, but this is the commit we just created in the second repo, and we can see that file ack has the new content.

Switching back to the first repo, we sync with the remote by running:

jj git fetch

Running jj log, we can see vvunluys now matches the commit we pushed from the other repo, but a bit confusingly, jj has automatically moved us to a new empty child of main. As a general rule, a new empty change is created any time the working copy commit is abandoned, and you can see in the fetch command output that jj considers the old commit abandoned. In truth, though, the prior commit was actually just replaced in the fetch by a new commit sharing the same change id and bookmark, so arguably this behaviour isn’t very sensical. 

In any case, we can easily fix the situation by switching back to the bookmarked change. Doing this will actually automatically abandon the empty change, because as another somewhat confusing rule, jj automatically abandons any empty change when you switch away from it, but only if the empty change has no children.

Anyway, you should now have some concrete idea of how to work with a remote repo.


SPLIT PR

For our last demonstration, let’s see how we can split a PR into smaller PRs and keep them in sync, similar to the stacked diff workflow in git.

So let’s say that we first want to move some changes from our PR change into a new child change that will become its own PR. One way to do this conveniently is with the command jj split. With no arguments, jj split runs in interactive mode, presenting us with an interface to select changes, and once we’ve confirmed the selection, jj moves the selected changes into a new child. 

The new child has id tlrvstwtw. If we switch back to vvunluys, you can see it no longer has the changes we moved into the new child.

A bit annoyingly, though, jj has moved our bookmark to the new child, which sort of makes sense because the child reflects the full state before we made the split. Still it’s confusing that the bookmark name no longer matches the id of the change it points to. To fix this, we could either move the bookmark or rename it. Either way will require a bit of hassle when we create the additional PRs and give them accurate descriptions, but moving the bookmark works out a bit simpler (for one thing, it avoids us having to rename the remote branches):

jj bookmark move –from tlrv –to vvun –allow-backwards
Now let’s push the new child with its own tracked bookmark and then make a PR. So first:

jj git push -c tlrv

When we create a PR for this new child, we again target main as the base branch. 

So now we have two separate PRs tracked by two separate bookmarked changes in our jj repo. Thanks to jj’s ability to easily edit stacked changes, any edits we make in the parent will automatically propagate into the child. Any time we need to sync, we can run jj git push with argument --tracked, which pushes new commits for all of the tracked bookmarks.

For an example, let’s say a PR reviewer asks for a change in the parent PR that necessitates modifying file ack. In the jj repo, we simply switch to the parent revision, modify the file, commit, check that we haven’t introduced any conflicts in the child, and then push all tracked bookmarks.

By the way, until the parent PR has landed, the github diff shown for the child PR will by default include all of the parent’s changes, not just the child’s. To show only the child’s changes relative to the parent, we can select just the child commit in the github interface.

Now what about the more general case, where you want to move changes between existing changes, such as between parent and child? Well whereas the jj split command creates a new revision, the jj squash command will simply move changes from one revision to another existing revision. (The command is called squash, by the way, because it by default moves changes from parent to child, which is how you generally squash a chain of commits into one.)

The file’s whose changes you wish to move can be specified in the command, or you can use -i to pick the changes interactively, which also lets us pick the specific edits within the files.

Let’s first try moving the addition of file ack from the parent to the child:

jj squash --from vv --to tl ack 

Then to move it back, we simply flip the –from and –to targets:

jj squash --from tl --to vv ack 

To understand splitting and squashing a bit more clearly, let’s restore back to the state before we split our PR. Looking in the operation log, we find the state id of the prior operation, and then we run:

jj operation restore <id>

Let’s now split this change again, but instead of splitting the changes into a new child, let’s split them into a new parent. We can do that with jj split again but this time use the --insert-before flag, specifying @ as the commit we want to give a new parent.

Notice how, this time, the split command did not move our bookmark. This small tweak usually makes more sense for splitting up a PR because the original PR state normally reflects the desired end state, so it should be the last PR in the chain.

Also, note here that we don’t strictly need the split command at all. We could instead just use jj new to insert a new empty change anywhere we like in the history, and then we can use jj squash to move changes from any revision to any other revision. So jj squash is effectively just a convenience:

jj operation restore <id>

jj new --insert-before @

jj squash -i –from @+ –into @

@+, by the way, is shorthand for the immediate children of the current commit. In this case, the current commit has just one child, so we are moving the changes into just one child.

Now, very last thing, what about the simple case where you want to push and pull main directly?


Hopes for future jj developments:
- native backend (avoid mental overhead of syncing with a git repo)
	- would this native backend still be able to interop with git?
- redesign / rename some commands
- fix the confusing use of the term “change”

[in log, what does blue diamond marker mean?]

