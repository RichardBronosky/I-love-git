# Git Hash Demo

This demonstrates how git commit hashes are calculated.

## Check Point 1

The first 3 lines of this file were in the first commit. Every other commit will
add an `H2` stating the "Check Point" number I am refering to. (I will also tag
the previous commit to make it easy to `checkout` the code in a state that will
allow you to run the commands yourself and get the same result.

What we really care about is the commit hash. Let's look at how it's made.

### A Tale of Two Hashes

Comparing the output of the porcelain command `git log` and the plumbing command
(you've probably never heard of) `git cat-file`. You will notice that in
addition to the "`commit` hash" that we always refer to, there is also a "`tree`
hash" and both are 40 heximal digits long. They are similar because they are
both `SHA1` sums.

```
$ git log
commit dbe1e878f2e19c2f5b6a4267780927e051a2f4af (HEAD -> master, tag: cp1)

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
`commit` type object dbe1e878f2e19c2f5b6a4267780927e051a2f4af. But you'll notice
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

## Check Point 2

In the previous commit, I called that metadata, a "file". We can locate the file
.in the `.git` folder by searching for files with a name ending with the last
few characters of the hash.

```
$ find .git -name '*a2f4af'
.git/objects/db/e1e878f2e19c2f5b6a4267780927e051a2f4af
```

We can check the log to see the new hash.

```
git log
commit 5f0a3dea2d90f4faf46695f53745e604994f3f6d (HEAD -> master)
Author: John Doe <john@example.com>
Date:   Fri Apr 16 00:55:07 2021 -0500

    Explain the commit metadata

commit dbe1e878f2e19c2f5b6a4267780927e051a2f4af
Author: John Doe <john@example.com>
Date:   Fri Mar 19 08:46:57 2021 -0500

    Initial Constitution
```

We can get the metadata for that commit.

```
$ meta
tree 36696c6a107ce4afa87b45ea43feee003b248097
parent dbe1e878f2e19c2f5b6a4267780927e051a2f4af
author John Doe <john@example.com> 1618552507 -0500
committer John Doe <john@example.com> 1618552507 -0500

Explain the commit metadata
```

This, being **not** the first commit, has an additional `parent` line in it. If
we check the **actual** `git` log file, we will see that this data is
derivitive.

```
$ cat .git/logs/HEAD
0000000000000000000000000000000000000000 9d3b7b986fc7c9c83f99d51162918f1741b60da2 John Doe <john@example.com> 1616161616 -0500	commit (amend): Initial Constitution
dbe1e878f2e19c2f5b6a4267780927e051a2f4af 5f0a3dea2d90f4faf46695f53745e604994f3f6d John Doe <john@example.com> 1618552507 -0500	commit: Explain the commit metadata
```

Here we can see that every line in the log has:
- The hash that our HEAD was at before
- The hash that our HEAD was at after
- Who performed the action
- When the action was performed
- What the action was
- The *summary* of the message give

Before I talk about what I mean by "summary", lets look at what else is in that
logs directory.

```
$ find .git/logs -not -type d
.git/logs/HEAD
.git/logs/refs/heads/master
```

Both of those are files. I filtered out the directories because it let's me look
cool when proving my next point.

```
$ sha1sum $(!!)
edec300e7d91d06ad48dcbeb0b637224da1e3231  .git/logs/HEAD
edec300e7d91d06ad48dcbeb0b637224da1e3231  .git/logs/refs/heads/master
```

Rather than `cat`ing the master log file to prove that the contents look the
same, I proved that the contents are cryptographically congruent by hashing
their content with the command used to produce a `SHA1` sum. If you can produce
that from the content of a file, you can create the hash yourself.

To do that, it will be easier to read, if we *write* a few more tools.

```
length(){ printf "commit %s\0" $(meta | wc -c); }
```

We will use the `length` function to prepend the "file size" to the `meta`
function and the `sha1sum` of that will match our commit hash.

```
$ length
commit 230

$ length; meta
commit 230tree 36696c6a107ce4afa87b45ea43feee003b248097
parent dbe1e878f2e19c2f5b6a4267780927e051a2f4af
author John Doe <john@example.com> 1618552507 -0500
committer John Doe <john@example.com> 1618552507 -0500

$ (length; meta) | sha1sum
5f0a3dea2d90f4faf46695f53745e604994f3f6d  -
```

But if I copy the output of from my screen and get the `sha1sum` of that, it
will **not** match our commit hash.

```
$ pbpaste | sha1sum
141c5d133bb8fe58548e53395801202bdab166e0  -
```

The reason for this is because even though it looks like `commit 230` is right
up against `tree` in the output. If we look carefully at our `length` function,
we'll notice that `commit %s\0` puts a `null` byte character at the end. But, we
can't see it on our terminal. We can't copy it. But, we can see that changing a
single byte, which was imperceivable to our eyes, results in a hash which is
**very** easy to recognize as *broken*. That is why I didn't cat the
`.git/logs/refs/heads/master` file earlier to compare it to the
`.git/logs/HEAD`.
