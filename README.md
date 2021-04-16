# Git Hash Demo

This demonstrates how git commit hashes are calculated.

## Second Commit

The first 3 lines of this file were in the first commit. Every other commit will
add an `H2` stating the commit number. But what we really care about is the
commit hash. Let's look at how it's made.

### A Tale of Two Hashes

Comparing the output of the porcelain command `git log` and the plumbing command
(you've probably never heard of) `git cat-file`. You will notice that in
addition to the "`commit` hash" that we always refer to, there is also a "`tree`
hash" and both are 40 heximal digits long. They are similar because they are
both `SHA1` sums.

```
$ git log
commit 8a434dfee44117b0048398957d24e811c8ba2a20 (HEAD -> master)
Author: John Doe <john@example.com>
Date:   Fri Mar 19 08:46:57 2021 -0500

    Constitution

$ git cat-file commit HEAD
tree 2c6584c6e3037c52154af0e1a01020f22b77feb7
author John Doe <john@example.com> 1616161617 -0500
committer John Doe <john@example.com> 1618548359 -0500

Constitution
```

The `cat-file` subcommand of the `git` command is doing exactly what it sounds
like. It's `cat`ing and file. What file is that? The metadata file for the
`commit` type object 8a434dfee44117b0048398957d24e811c8ba2a20. But you'll notice
I didn't call it by its hash. I called its alias -- that's "alias" in the
"legal" definition, **not** the "computing" definition -- `HEAD`. The `log`
output above shows that there is another alias `master` (which is the name of
the branch). And another alias `cp1` which is a tag. I could have used any of
those *references* to the object.

To make the code more readable in the future steps, I'm going to define a shell
function.

```
meta(){ git cat-file commit "${1:-HEAD}"; }
```

Rather than simply hardcoding `HEAD` into the function, I used the `:-` modifier
to provide a default value for the argument `${1}` when it is not passed. See:

```
$ meta
tree 2c6584c6e3037c52154af0e1a01020f22b77feb7
author John Doe <john@example.com> 1616161617 -0500
committer John Doe <john@example.com> 1618548359 -0500

Constitution
```
