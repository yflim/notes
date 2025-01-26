
# Table of Contents

1.  [Config](#orgee73177)
2.  [Adding & Committing](#orge6fc301)
3.  [Rebasing](#org004821d)
4.  [Diffs](#org7009344)
5.  [Logging / History](#orgfa35753)
6.  [Time Travel](#orgd0bc200)
7.  [Rewriting History](#org92772a1)
8.  [Checking Out](#orgdd8922d)
9.  [Cloning](#org71163ba)
10. [Pushing](#orgd05a9fe)
11. [Branching](#org57397bb)
12. [Fetching](#orge5b91e0)
13. [Stashing](#orgd5615c0)
14. [Recommended Workflow](#org94cd451)
15. [Undoing push to remote](#org6d67f12)
16. [Pull from remote and override local](#orgfc7f7d2)
17. [Help](#org4ca9ce7)
18. [Remotes](#org546051c)
19. [GitHub](#org4e5d5f3)
20. [First Aid](#org2aa971b)
21. [Utils](#org830d28d)


<a id="orgee73177"></a>

# Config

`git config --list`

Modifying .gitconfig:
`git config --global color.ui false`
yields:

    [color]
            ui = true


<a id="orge6fc301"></a>

# Adding & Committing

to start tracking an existing project, go to the project’s directory and
`git init`
Add all files in .
`git add .`
`git status`
To remove from tracking? (how to unstage?):
`git rm --cached <file>...`
commit: -v for verbose.
`git commit -m "Initial commit"`
Amend commit message (for unpushed last commit):
`git ci --amend`
`git cherry-pick commit`
Cherry-pick all commits after A up to and including B:
`git cherry-pick A..B`
Cherry-pick all commits after A up to and including B:
`git cherry-pick A^..B`

apply changes to a specific file f in branch B to A:
`git co A`
`git co --patch B f`

Listing tags:
`git tag`
Listing tags matching a pattern:
`git tag -l "v1.8.5*"`
Annotated tag on current commit:
`git tag -a v1.4 -m "my version 1.4"`
Lightweight tag on current commit:
`git tag v1.4-lw`
Push tags:
`git push --tags`
Find Git short SHA of tag in local repo:
`git rev-parse --short v0.0.1`
Find Git short SHA of tag in remote repo:
`git ls-remote https://github.com/yourname/time-lib.git v0.0.1`


<a id="org004821d"></a>

# Rebasing

rebase experiment onto (after) commits in master since most recent common ancestor:
`git checkout experiment`
`git rebase master`
`git checkout master`
`git merge experiment`

rebase in interactive mode starting at commit preceding <commit>:
`git rebase -i <commit>~1`
rebase in interactive mode starting at commit after <commit>:
`git rebase -i <commit>^`
Following the above, to modify a specific commit, replace &ldquo;pick&rdquo; with &ldquo;edit&rdquo; in the relevant line, quit editor, then follow instructions.

rebase last 4 commits in interactive mode
`git rebase -i HEAD~4`

rebase a topic branch based on one branch to another (specify topic if not on branch):
`git rebase --onto newbase oldbase [topic]`


<a id="org7009344"></a>

# Diffs

Show all differences between master & branch - everything on each that isn&rsquo;t on the other:
`git diff [--stat] master branch`
Show changes in branch from master since the common commit they diverged from - only what&rsquo;s on branch that isn&rsquo;t on master:
`git diff [--stat] 8d585ea branch`
Let git figure out the last common commit and do the diff off it:
`git diff [--stat] master...erlang`

Difference between modified copy and head:
`git diff -- file HEAD`
Difference between commit and head:
`git diff commit-hash [HEAD] [filename]`
Difference between commit and parent:
`git diff commit-hash^ commit-hash [filename]`
or
`git diff commit-hash^! [filename]`
Difference between commits 2 and 3 times back
`git diff HEAD~3 HEAD~2 [filename]`

Difference between version in tip of branch and specified file:
`git diff te-staging -- filepath`
Compare file between branches:
`git diff branch1 branch2 path/to/file`

Commits on remote branch only:
`git log HEAD..origin/<branchname>`

Show staged changes:
`git diff --staged readme.txt`
or:
`git diff --cached`


<a id="orgfa35753"></a>

# Logging / History

`git log [remotename/branchname]`
Sort of equivalent to - git show SHA:
`git log -p`
Show commit history of file:
`git log -- file`
Show file history, using patches (-p) and showing only commits that are enough to explain how the file came to be (&#x2013;):
`git log -p -- file`
Concise version of -p, showing additions & deletions:
`git log --stat`
Show history:
`git log --graph --oneline --decorate --all`
\\\* 594f90b (HEAD, tag: v1.0, master) reverted to old class name

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">\* 1834130 (erlang) added haskell</td>
</tr>


<tr>
<td class="org-left">\* ab5ab4c added erlang</td>
</tr>
</tbody>
</table>

\\\*   8d585ea Merge branch &rsquo;fix<sub>readme</sub>&rsquo;
&#x2026;
List commits on oldbranch but not newbranch:
`git log oldbranch ^newbranch`

Show branches containing commit:
`git branch --contains commit`

Get last common commit of master and branch:
`git merge-base master branch`

Show parent hashes for a commit
`git rev-list --parents -n 1 <commit>`

Show the log message and textual diff. Also presents the merge commit in a special format as produced by git diff-tree &#x2013;cc.
`git show commit-hash`
Concise show:
`git show --stat commit-hash`
Show commit that&rsquo;s parent of HEAD
`git show HEAD^1`
Show commit that&rsquo;s grandparent of HEAD
`git show HEAD^2`
Show commit that&rsquo;s great-grandparent of HEAD
`git show HEAD~3`

Show version of file at specific commit
`git show REVISION:path/to/file`

Search commits on all branches for file:
`git log --all -- <path>`
where, e.g. path = app/views/terms/edit.js.erb

To find all commits where number of occurrences of &ldquo;word&rdquo; changed in file contents:
`git log -Sword`
To find all differences whose added or removed line matches *word*, not necessarily with the # of occurrences of *word* changed:
`git log -Gword`
Or, to limit the above to a particular file, e.g.:
`git log -S "string" -- <file-path>`
And to show only the relevant changes in that file:
`git log -S "string" -p -- <file-path>`

Limit the commits output to ones with log message that matches the specified pattern (regular expression).
`git log --grep=<pattern>`

Show commits on date(s):
`git log --after="2016-04-04" --before="2016-04-08"`

Display version of file at particular commit:
`git show <SHA> -- <path-to-file>`

Show all commits, even those no longer accessible directly on branches:
`git reflog`
`git log -g`


<a id="orgd0bc200"></a>

# Time Travel

Reset file to particular commit:
`git co <hash> <filename>`


<a id="org92772a1"></a>

# Rewriting History

Undo commit while retaining subsequent ones, but keep in log the revert (undoing):
`git revert --strategy resolve <commit>`


<a id="orgdd8922d"></a>

# Checking Out

e.g. to fix an accidental delete:
`git co -f`

Create a remote-tracking branch:
`git checkout -b [branch] [remotename]/[branch]`
Shorthand for previous:
`git checkout --track origin/branch`


<a id="org71163ba"></a>

# Cloning

`git clone url [dstdir]`
Clone only one branch:
`git clone [-b branchname] --single-branch remote-uri`


<a id="orgd05a9fe"></a>

# Pushing

`git remote add origin git@github.com:<username>/first_app.git`
`git push -u origin master`
-u for upstream: track master if current branch doesn&rsquo;t already?

Push all commits up to and including chosen commit to remote:
`git push <remotename> <commit SHA>:<remotebranchname>`


<a id="org57397bb"></a>

# Branching

Show branches (-r for remote):
`git branch [-r]`
Ordered by most recent commit:
`git for-each-ref --sort=-committerdate refs/heads/`

Rename branch:
On current branch:
`git br -m <newname>`
From another branch:
`git br -m oldbranch newbranch`

Create a remote-tracking branch:
`git checkout -b [branch] [remotename]/[branch]`
Shorthand for previous:
`git checkout --track origin/branch`

Track a remote branch with an existing local branch:
`git br -u origin/branch`

Create a new remote branch:
`git co -b branch`
`git push --set-upstream <remote-name> <local-branch-name>[:<remote-branch-name>]`

Pull a remote branch:
`git br remote_branch_name origin/remote_branch_name`
`git co remote_branch_name`

Delete a branch:
`git br -d <branch>`

Delete oldname remote branch and push newname local branch:
`git push origin :oldname newname`


<a id="orge5b91e0"></a>

# Fetching

`git fetch [remote] [repo/branch]`
`git fetch --all`
`git fetch --dry-run`

`git fetch origin`
`git log --oneline main..origin/main`
`git co main`
`git mg origin/main`


<a id="orgd5615c0"></a>

# Stashing

Stash only a single file:
`git stash -- file`

`git stash list`
`git stash apply [stash-name]`
Apply only tries to apply the stashed work — you continue to have it on your stack.
To remove, git stash drop with the name of the stash to remove:
`git stash drop <stash-name>`

Show stash contents:
`git stash show -p <stash-name>`

Clear stash:
`git stash clear`


<a id="org94cd451"></a>

# Recommended Workflow

List branches:
`git branch`
Check out, create new branch, and switch to it:
`git co -b modify-README`
[- git branch]
Rename file, in this example; result doesn&rsquo;t count as new file to git
`git mv README.rdoc README.md`
Make changes&#x2026;
Instead of git add ., -a to commit all modifications to existing files
`git commit -a -m "Improve README"`
`git co master`
`git merge modify-README`
Optionally delete branch:
`git br -d modify-README`
To abandon topic branch changes; -D deletes even if/though changes haven&rsquo;t been merged:
`git br -D modify-README`
`git push`


<a id="org6d67f12"></a>

# Undoing push to remote

Approach 1:
`git revert <commit-hash>`
where <commit-hash> can be HEAD.
Then commit and push, I assume?

Approach 2:
`git push -f origin <desired-commit-hash>:<branch-name>`
or
`git reset --hard <desired-commit-hash>`
`git push origin -f`
where <desired-commit-hash> can be HEAD^ (parent of HEAD).

In case of commit being a merge: If 2nd approach doesn&rsquo;t work, trickier, need to Google,
but hope the unlucky event doesn&rsquo;t happen to begin with.


<a id="orgfc7f7d2"></a>

# Pull from remote and override local

`git fetch remote`
`git reset --hard remote/branch`


<a id="org4ca9ce7"></a>

# Help

`git help <verb>`
`git <verb> --help`
`man git-<verb>`


<a id="org546051c"></a>

# Remotes

View existing remotes (-v shows URLs used for fetch and push)
`git remote [-v]`
Add remote:
`git remote add name url`
Change the &rsquo;origin&rsquo; remote&rsquo;s URL
`git remote set-url origin https://github.com/user/repo2.git`
Show remote info:
`git remote show <remote-name>`
Set local branch to push to tracked remote branch by default:
`git cfg push.default tracking`
Delete remote branch:
`git push <remote_name> --delete <branch_name>`
Prune all stale tracking branches:
`git remote prune origin`


<a id="org4e5d5f3"></a>

# GitHub

Set up new repo:
`Create new repo on GitHub (click "+" drop-down menu on top right, then create without any files)`
`cd project-dir`
`git init`
`git add .`
`git ci -m "first commit"`
`Copy repo URL`
`git remote add origin remote-repo-url`
`git remote -v`
`git push -u origin master`


<a id="org2aa971b"></a>

# First Aid

Recovery after losing a stash:
Find object ID of dropped stash in output from stash pop
`git stash pop`
[&#x2026;]
Dropped refs/stash@{0} (2ca03e22256be97f9e40f08e6d6773c7d41dbfd1)]
Get stash back (as a branch)
`git br tmp 2ca03e`
Convert this to a stash
`git stash apply tmp`
`git stash`

Or, better yet(?), apply the stash from the hash value:
`git stash apply 2ca03e`


<a id="org830d28d"></a>

# Utils

Get the current commit (I can&rsquo;t parse this `rev-parse` documentation; &ldquo;porcelainish&rdquo;&#x2026; what the fuck is that???)
`git rev-parse --short HEAD`

