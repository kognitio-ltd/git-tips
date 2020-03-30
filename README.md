# Ben's Git tips

This is a collection of mostly intermediate-level tips and tutorials I
have found since starting to use git.

### Multiple remote repositories

Perhaps you have a repo with a remote called "origin" but you want to fetch changes from a different repo.  You need to add the second repo manually:

 1. Add the second repo as a remote called "reponame"
    ```
    git remote add reponame git@git:wx2_origin  # or a repo directory, e.g. /path/to/repo
    ```

 2. List the remotes it knows about: it should have the original repo and the one you added.
    ```
    git remote -v
    ```

 3. Fetch the branch `remote_branch` from reponame and call it `local_branch` locally.  (E.g. both `local_branch` and `remote_branch` might be called `VERSION_80100`.) 
    ```
    git fetch reponame remote_branch:local_branch
    ```
    Now you can switch to `local_branch` or cherry pick from it.

 4. You can then pull changes from the remote or push your changes using:
    ```
    git pull origin      # Pull from "origin"
    git push origin
    git pull reponame    # Pull from "reponame"
    git push reponame
    ```

### Diff including changes you have staged

If you have staged changes using `git add` then they stop showing up in `git diff`.  To see them use:

```
git diff --cached
```

### Search for a commit with a message/patch containing a string

Search for commits whose messages match the regexp `<foo>`:

```
git log --grep=<foo>
```

Search for commits whose patch (changes added or removed) matches the regexp `<bar>`:

```
git log -G<bar>
```

### Ignore whitespace changes

The `-w` option to `git show`, `git diff` and `git blame` ignores
whitespace changes, which can considerably simplify the output.

### Switch to most recent branch

`git checkout -` will switch to the previous branch you were on, so you
can you it to switch between two branches.   (This is just like `cd -`
but for branches instead of directories.)

### Referencing a commit by regexp

You can reference a git commit using `:/<regexp>`, a regexp search for
the pattern `<regexp>` in the commit texts (not just the title) for the
*youngest matching commit which is reachable from any ref.*  Note that
this might be in a different branch, and it will be the newest one if
there are multiple matches.  E.g. `git show :/IS_NUMBER`.  (See `git
help revisions`.)

### Git bisect following one branch

To do git bisect on just the first parent, mark the other parents as
bad: see https://stackoverflow.com/a/5652323/ for details.

### Git bisect within one commit

If a commit is too complicated you can split it up and bisect individual
chunks: see https://gist.github.com/wisq/0fa021df52a3bd2485ac for
details.

(You might find that most of the partial commits don't compile.  If so,
try to separate out the header file changes, or whatever, as necessary
first.)


### Patching from another repo

If you want to take changes from another repo but don't want to add it
as a remote you can add changes using patches.

NB The following require the `git apply` to be called from the top-level
directory of the destination repo otherwise it will [fail silently](https://stackoverflow.com/questions/24821431/git-apply-patch-fails-silently-no-errors-but-nothing-happens).

To create a patch:

```
cd sourcerepo && git format-patch <commit1>..<commit2> > patchfile
```

To apply a patch:

```
cd destrepo && git apply patchfile
```

Do both in one command:

```
(cd sourcerepo && git format-patch --stdout <commit1>..<commit2>) | (cd destrepo && git apply -)
```

For uncommitted changes;

```
(cd sourcerepo && git diff) | (cd destrepo && git apply -)
```

### I want to have several checked out trees but one git repo

Use [`git worktree`](https://spin.atomicobject.com/2016/06/26/parallelize-development-git-worktrees/).


### Pushing excluding the last commit(s)

Perhaps you have commits that you want to push but the most recent one
is experimental and you want to hold back on it.

```
git push origin HEAD^:<branch_name>
```

In general to push everything up to and including some earlier commit:

```
git push origin <commit>:<branch_name>
```

### The merge results are too confusing

When doing a merge these special filenames become available to `git
show` and `git diff`:
 * The common ancestor is `:1:<filename>`
 * The `HEAD` version is `:2:<filename>`
 * The `MERGE_HEAD` version is `:3:<filename>`

### I'm resolving a merge conflict but munged the file

If you edit a file with conflict markers (`>>>>>>>`, etc.) to resolve a
merge conflict but mess it up you can [ask git to re-create
it](https://gitster.livejournal.com/43665.html) using:

```
git checkout -m <filename>
```

### Git reset and reflog

To deal with e.g. an undesired merge and push:
 * Before doing any of these commands use `git status` to check you haven't got any uncommitted changes.  If you have, or in any case, stash them and/or take a backup.
 * You can use `git reset --hard <ref>` to move the current branch (and HEAD) to point to a different commit `<ref>`, in this example (eventually!) the one before the merge.
 * You can use `git log` to see commit refs for commits on the current branch that you might want to move back to.
 * Or you can use `git reflog` which shows all the previous commits HEAD has pointed to, which might not be on the current branch (e.g. if you've just done a regrettable rebase or reset).
 * If you have rewritten history before the origin's HEAD it won't let you push so you need to use `git push -f <origin> <branch>`.

### Git has forgotten renames after "git stash pop"

See http://stackoverflow.com/questions/8495103/git-stash-and-pop-shows-file-no-longer-marked-as-moved

```
git rm --cached file1
```

### I've popped and lost (or dropped) a git stash and want it back

Look at these commits, which are the dangling commits (not yet garbage
collected) for ones with a title like
`WIP on <somebranch>: <commithash> <commit message>`.

```
git fsck --no-reflog | awk '/dangling commit/ {print $3}'
```

This is more pleasant using gitk:

```
gitk --all $( git fsck --no-reflog | awk '/dangling commit/ {print $3}' )
```

Then you can use `git stash apply <hash>` to apply it (or perhaps `git
stash store <hash>` to save it).

### I want my repo to be readable by people in my group or everybody

Set git config option core.sharedRepository to group or all.  For
gitolite 3 use $UMASK.  See
https://stackoverflow.com/questions/7086325/setting-umask-in-git-gitolite

### Git thinks a file has changed but you don't agree

Perhaps you're trying to switch branch and git says you can't because a
file has changed, but `git diff` doesn't show any differences and `git
stash` says nothing has changed.  

This can happen if the file has a filter on it and its timestamp has
changed, and it is different from the version in the repo although the
"clean" filter version isn't different.  If you're sure then fix using:

```
git checkout -f <file>
```

### I want to display git context in my prompt

Download
[git-prompt.sh](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh),
if you don't already have it as part of your git distribution, and
follow the instructions in the comment at the start.

### General references and tutorials

 * Pro Git book [website](https://git-scm.com/book/en/v2), [GitHub page](https://github.com/progit/progit2) and [PDF](https://github.com/progit/progit2/releases/download/2.1.77/progit.pdf)
 * [Git for Ages 4 And Up](https://www.youtube.com/watch?v=1ffBJ4sVUb4)
 * Explanation of git [remote branches](https://blog.plover.com/prog/git-remote-branches.html) and [rejected push error](https://blog.plover.com/prog/git-ff-error.html)

### Problem-fixing tutorials

Every change that git is told about, including "git add" and "git stash", is saved in git.  You should be able to recover from most mistakes using "git reflog" and "git show" together with "git reset", "git checkout" or "git cherry-pick".

 * [Reflog, your safety net](http://gitready.com/intermediate/2009/02/09/reflog-your-safety-net.html)
 * [Recovering lost commits with git reflog and reset](http://effectif.com/git/recovering-lost-git-commits)
 * [Use "git reflog" and "git cherry-pick" to restore lost commits](http://www.ocpsoft.org/tutorials/git/use-reflog-and-cherry-pick-to-restore-lost-commits/)
 * [On undoing, fixing, or removing commits in git](https://sethrobertson.github.io/GitFixUm/fixup.html) in the style of "choose your own adventure" 
 * ["Flight rules" for Git](https://github.com/k88hudson/git-flight-rules/blob/master/README.md#flight-rules-for-git)
 
### Tutorials on Git internals

 * [Disecting Git's Guts](https://youtu.be/YUCwr1Y6bFI)
 * Git from the Inside Out [essay](https://codewords.recurse.com/issues/two/git-from-the-inside-out) and [talk](https://maryrosecook.com/blog/post/git-from-the-inside-out-talk)
 * [Advanced Git: Graphs, Hashes and Compression, Oh My!](https://www.youtube.com/watch?v=ig5E8CcdM9g)
 * [Git from the bottom up](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf)
