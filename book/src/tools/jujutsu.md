# Introduction to Jujutsu Version Control

*This text is a supplement to a [video about the Jujutsu version control system](https =//youtu.be/mM4nrhDenC8).*

## Pros and cons

*Jujutsu*, otherwise known as *JJ*, is a version control system that has a number of notable advantages over Git =

1. JJ commit history can be (psuedo-) mutated, even though the commits themselves are strictly immutable. Mutable history makes certain workflows much easier, such as [stacked diffs](https =//newsletter.pragmaticengineer.com/p/stacked-diffs).
1. JJ stores conflicts as commit meta-data, which can make conflicts easier to resolve in merges.
1. JJ has no concept of an index, so commits do not have to be staged. Instead, running any JJ command will commit any dirty changes in your working copy before doing anything else. Effectively, you never need to stash, and you never get stuck in the middle of a merge = you can always just switch away to any branch at any time without losing any work.

On the other hand, Jujutsu arguably has some drawbacks =

1. JJ is still pre-1.0 release. Though generally reliable already, the command line interface has not been entirely stable.
1. JJ doesn’t currently support git-lfs, so it may not be a practical choice for projects with numerous or large binary files.

## Backends and Git interop

JJ is architected to support swappable storage backends, but the only backend fully supported at the moment is Git itself. When using this backend, a `.jj` repo directory and a `.git` repo directory sit side-by-side in the root of your working copy, and every JJ commit is stored as a Git commit, along with some additional JJ meta-data. Likewise, the full command history of your JJ repo is stored as additional Git commits.

When cloning from or synching through a remote repo, the remote is just a regular Git repo with no knowledge of JJ. The Git commits contain all the JJ meta-data, so JJ users can sync just with `push` and `fetch`. In fact, you can clone any Git repo and work with it on your own as a JJ repo, even if other users of the Git repo do not.

## Data model

The JJ model differs from Git in several ways =

- JJ **commits can be hidden** = every JJ commit has a visibility flag, and various commands will toggle this flag. What it means to be hidden is simply that hidden commits will be omitted by default when displaying history with commands like `jj log`. This is helpful for users because it removes clutter from the history, particularly when old commits get logically replaced by newer versions.
- The **operations log** is an immutable history that tracks every command which modifies the state of the repo. With each command, the log records the set of commits that were visible after the command executed, and this allows the repo back to be easily and quickly restored back to any prior state. The commands which restore an old state are themselves appended to the log and never actually create or delete any commits = instead, an old state is restored simply by toggling which commits are visible.
- In addition to the normal Git content hash commit ids, JJ commits also have a **change id**. These change ids are represented with only lowercase English letters, and they are either randomly generated or inherited from a prior commit (depending upon the operation that creates them). Most of the time, a repo will have one visible commit at a time per individual change id, but there are scenarios where a repo may have multiple visible commits simultaneously for an individual change id. This situation is called a “divergent change”. While divergent changes are not error states, *per se*, they do make your commit history a bit confusing, so normally you’ll rectify the situation in one of a few ways, such as by hiding all but one of the commits or maybe by merging the divergent changes together.
- JJ can store **conflict meta-data** in commits. For example, if a commit has two parents with a conflict in a certain file, the irreconcilable difference is stored in the commit alongside each parent’s version of the file. Commits with these conflicts are marked in the log. Like divergent changes, conflict commits in your repo are not an error state, *per se*, but normally you’ll want to rectify the situation by making new child commits that resolve the conflicts, or alternatively, you might simply hide a commit with conflicts (which may be appropriate if you just want to abandon a merge).
- Instead of branches, JJ has what it calls **bookmarks**, though they are basically the same thing. The main difference is that JJ has no concept of a 'current bookmark', and bookmarks do not automatically advance the way Git branches do. It’s common in JJ to locally track different branches of work by change ids rather than with bookmarks, so bookmarks are mostly used in JJ just to sync with remote branches. If a bookmark that tracks a remote branch somehow ends up out of sync with the remote branch, say because the bookmark was manually moved, then the bookmark becomes conflicted. These conflicts can be resolved by simply getting the bookmark and its tracked remote back in sync.
- Whereas a Git commit can have at most two parents, a JJ commit can have **any number of parents**. More than two parents can be useful for cases where you want to do a multi-way merge = instead of having to merge multiple pairs, you can just merge everything together directly in one operation. Along with JJ’s conflict meta-data, this can help reduce the number of conflicts you must manually resolve.

## Demo of basic operations

```
$  mkdir test-proj
$  cd test-proj

$  jj git init              # create .jj and .git subdirs
$  jj log                   # view history
$  touch apple              # make new file
$  jj                       # commit (if dirty working copy)
$  jj log                   # show history
$  jj edit 5d11             # switch to commit with id 5d11…
$  touch orange             # create a new file
$  jj edit a94              # commit working directory, then switch 
                            # back to commit with id a94…

```

> [!NOTE]
> Note that the `jj git` subcommand has several of its own subcommands for working with Git remotes and the underlying Git repo, but despite some of these subcommands sharing the same name as Git subcommands (such as `jj git init`, `jj git clone`, `jj git fetch`, and `jj git push`), you should think of these JJ Git subcommands as distinct from actual Git’s own subcommands. Also be clear that the underlying dot Git repo can be operated upon directly with Git just like any normal Git repo. Just keep in mind you may need to sync changes made directly through Git back to the JJ repo with the command JJ Git import. Normally though, you’ll manage a JJ repo through JJ itself rather than use Git directly.

Unlike Git, JJ does not have a concept of a current, checked out branch = instead, there is simply a current commit, and when we want to switch our working copy to another commit, we specify a commit id with the `jj edit` subcommand. So say, assuming there is a commit id that uniquely starts with `5d11`, we can switch our working copy to this commit with the command `jj edit 5d11`

To create a commit, we simply make changes to our working copy and then run JJ. *Any* time JJ runs, no matter the subcommand, it will make a new commit for any dirty changes in the working copy directory. So say if we create a new file and then switch again to a different commit with `jj edit`, JJ will first make a new commit before switching. Effectively, no matter the state of your repo and working copy, you can always switch away at any time.

> [!TIP]
> If you don’t like any changes that get committed, there are commands to easily remove them from your visible commit history. However, if you want to completely remove certain commits from your repo, you may have to fall back on manipulating the underlying Git repo state directly wtih `git` commands.

## Most common commands

- `jj git init` = initialize a new local repo (creates both `.git` and `.jj` dirs)
- `jj git clone` = clone a git repo (creates both `.git` and `.jj` dirs)
- `jj git fetch` = copy commits from the remote
- `jj git push` = copy commits to the remote (possibly updates remote branches)

---

- `jj log` =  show repo history
- `jj new` = create a new commit with a new changeid
- `jj abandon` = remove a commit from history (*i.e.* hide the commit and revise its ancestors to remove its changes)
- `jj squash` = move file changes from one changeid to another existing changeid
- `jj split` = move file changes from one changeid to a new changeid

---

- `jj operation log` = show the operations log
- `jj operation restore` = restore repo state to the state immediately after a particular operation
- `jj operation undo` = restore repo state to the state immediately before the last operation

## Common points of confusion

### What is a change?

When we talk about a “change” in JJ, it’s not always clear if we’re talking about a change id or if we’re talking about a commit that has a particular commit id. More confusingly, the term “change” can also refer to the actual file content changes of a commit relative to its parents, and the JJ documentation is not always clear about this distinction. These ambiguities would have been avoided if “change ids” were instead named something else, like perhaps, “revision ids”, but as it is, we have to be careful when we say “change”.

### Incremental commits are replacements, not descendants

In typical Git usage, people typically create incremental commits as they work on a branch, and this creates a chain of parentage: each new commit is a child of the prior.

In JJ, incrementally commiting dirty working changes produces commits that logically replace the prior commit in the history: the new commit has the same changeid and parents as the prior commit, and the prior commit becomes hidden.

You can always recover hidden commits, so you shouldn't worry about unrecoverable state, but to create a sort of 'checkpoint' in your development, the usual practice is to run the `jj new` command. This will create a new commit that:

1. has a new, random changeid
1. is a child of the prior commit
1. is empty (represents no file changes relative to the parent)

In a sense, this marks in your histroy that you are starting on the next phase of development, such as fixing the next bug or implementing the next feature.

> [!NOTE]
> Pretty much all JJ commands create 'replacement' commits rather than extend the chain of parentage. The exceptions are the commands that create new changeids: `jj new` and `jj split`.

### Restoring old operations does not delete log entires or commits

When restoring an old repo state with the `jj operation restore` or `jj undo` commands, the visibility of relevant commits may be toggled, and these commands are appended to the operation log.

However, the only way operation log entries or commits get truly deleted from a repo is by running the `jj util gc` command. This command deletes entries from the operations log that are older than a certain threshold (default of 2 weeks) and also deletes the commits that were only relevant to the pruned entries.

## The current empty commit gets automatically hidden when switching away

If your current commit is empty (meaning it represents no file changes relative to its parents), JJ will hide the commit if you switch away to another commit. This may happen not just if you run `jj edit` but if you run any command that switches you to a different current commit.