This work is licensed under the Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License.

http://creativecommons.org/licenses/by-nc-sa/3.0/

# Git Hash

Learn about all the different types of objects that hashes in Git can represent.
Learn how it calculates the hashes.

Find out why rebasing changes your commit hashes and deviates your branches.

Understanding how Git works internally may help you better use git and understand what's going on when you get problems.

## Audience Experience

I'm going to assume you already know how to use Git and may have a mental image of how it fits together.

You should be familiar with the following commands:

```bash
git add Readme.md
git commit
git rebase
git log --graph
# * commit 80698fe28d5792e472d8159b88e2f865e5c2d015
# | Author: Jeremy Soller <jackpot51@gmail.com>
# | Date:   Tue Jan 16 09:12:11 2018 -0700
# |
# |     Update README.md
# |
# *   commit bb9995847198a07e888a0a44b950a2dacb69d747
# |\  Merge: f87fc65 91cf4fe
# | | Author: Jeremy Soller <jackpot51@gmail.com>
# | | Date:   Sun Jan 14 07:23:35 2018 -0700
# | |
# | |     Merge pull request #1133 from dogHere/master
# | |
# | |     add autopoint as dependencie for ubuntu
# | |
# | * commit 91cf4feadf650362df96c185749ca17b62bccaec
# |/  Author: dogHere <dogHere@tutamail.com>
# |   Date:   Sun Jan 14 20:50:35 2018 +0800
# |
# |       add autopoint as dependencie for ubuntu
# |
# * commit f87fc6584153ea959e420cd9c7b509ea256c0ede
#   Merge: 2dbdfd4 4a45dd4
#   Author: Jeremy Soller <jackpot51@gmail.com>
#   Date:   Tue Jan 9 07:31:36 2018 -0700
#
#       Merge pull request #1132 from fengalin/master
#
#       Docker: add autopoint
```

This "easy to use" abstraction is what's known as Git's Porcelain.

## Git Plumbing

> Git is fundamentally a content-addressable filesystem with a VCS user interface written on top of it

From the [Git Book](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain).

We're going to be looking inside the `.git` directory.

```bash
ls .git | cat
# branches
# COMMIT_EDITMSG
# COMMIT_EDITMSG~
# config
# description
# HEAD
# hooks
# index
# info
# logs
# objects
# refs
```

Especially into the object directory.

```bash
find .git/objects -type f
.git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df
.git/objects/fe/95731bff3edc4547c6a4a565f16b305a1b2c9b
.git/objects/5e/fd594d1829678e7357667410441c97610e62aa
.git/objects/f1/c2caa3fc8074d6a119f1131bfe535e9696a988
.git/objects/37/e5eef484d3933da4f1a641077fdff4f3c6fb54
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

## Git Blobs

We can see that when we add a file to git it gets assigned a hash and gets
stored within the .git directory.

```bash
git add Readme.md
cat Readme.md | git hash-object -t blob --stdin
# 0b869ad2a86ffd86ff435e5851ed46eb3f7ca3df
ls .git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df
# .git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df
```

So what's in this file?

```bash
cat .git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df
# incomprehensible binary
git cat-file -p 0b869ad2a86ffd86ff435e5851ed46eb3f7ca3df
# # Git Hash
#
# Learn about all the different types of objects that hashes in Git can represent.
# Learn how it calculates the hashes.
#
# Find out why rebasing changes your commit hashes and deviates your branches.
# ...
git cat-file -p 0b869ad2a86ffd86ff435e5851ed46eb3f7ca3df | wc -c
# 2047
cat .git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df | zlib-flate -uncompress
# blob 2047# Git Hash
#
# Learn about all the different types of objects that hashes in Git can represent.
# Learn how it calculates the hashes.
#
# Find out why rebasing changes your commit hashes and deviates your branches.
```

We can see that our content was stored as a `zlib` compressed file with a short
header.
The header is a string prefix that says it represents a blob and that the
contents is 2047 bytes in length.
We don't see it, but the header is concluded by a 0 value byte.

Where does the hash come from?

```bash
cat .git/objects/0b/869ad2a86ffd86ff435e5851ed46eb3f7ca3df | zlib-flate -uncompress | sha1sum
# 0b869ad2a86ffd86ff435e5851ed46eb3f7ca3df  -
```

The hash is the `SHA-1` digest of the uncompressed header and file.

A reminder of a couple desirable properties of hash functions:

_Deterministic_: The same input always results in the same digest.

_Second preimage resistance (weak collision resistance)_: Given an input, it is
computationally infeasible to find another input such that the hash function
returns the same digest.

In terms of Git, determinism means that the same file contents will always be
stored with the same hash.

In terms of Git, collision resistance means that it is extremely unlikely that
2 different files will have the same hash.

This allows Git to safely treat the hash as a key to the data.

## Git Trees

We've seen how Git stores the contents of files.

Q: But how does it store directories?

A: In trees!

```bash
ls -l example/
# total 8
# -rw-rw-r-- 1 chris chris   29 Feb 20 20:18 hello.txt
# drwxrwxr-x 2 chris chris 4096 Feb 20 20:19 more
git cat-file -p e260cef5a4af6b5d5de628c6b2a8b23c45d83f27
# 100644 blob 2ec3a82131ef44c87e225b106700408bd52b205f	hello.txt
# 040000 tree e442ba5dca7d166ec06686a09cebb01129f92f71	more
cat .git/objects/e2/60cef5a4af6b5d5de628c6b2a8b23c45d83f27 | zlib-flate -uncompress | xxd
# 00000000: 7472 6565 2036 3800 3130 3036 3434 2068  tree 68.100644 h
# 00000010: 656c 6c6f 2e74 7874 002e c3a8 2131 ef44  ello.txt....!1.D
# 00000020: c87e 225b 1067 0040 8bd5 2b20 5f34 3030  .~"[.g.@..+ _400
# 00000030: 3030 206d 6f72 6500 e442 ba5d ca7d 166e  00 more..B.].}.n
# 00000040: c066 86a0 9ceb b011 29f9 2f71            .f......)./q
cat .git/objects/e2/60cef5a4af6b5d5de628c6b2a8b23c45d83f27 | zlib-flate -uncompress | sha1sum
# e260cef5a4af6b5d5de628c6b2a8b23c45d83f27  -
```

Unfortunately, trees are a mix of binary and text, so they're harder to
illustrate.

We can see that for a directory, Git stores the contents.
Each tree can be made up of several other trees and several blobs.

We can see that for every entry Git stores the mode, the file name and the
hash.

File modes starting with 1 are blobs.
We can see that Git stores slightly different permissions to what's on disk.

Next comes the file name, which is a null terminated string.

Finally comes the hash of the linked object.
We have to look at the hexadecimal conversion for that.
It is 20 bytes long.

Question: Could a tree contain multiple references to the same blob?

Question: Could a tree contain multiple references to the same tree?

Question: What does it mean for the repository size if a file or directory is
copied?

Question: Could a tree contain a reference to itself?

![git tree diagram](https://git-scm.com/book/en/v2/images/data-model-2.png)

Image illustrating trees from the brilliant
[Git book](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects).

Question: How would changing the contents of `./bak/test.txt` affect `./`?

## Git Commits

Commits are probably where we're most used to seeing hashes.

```bash
cat .git/refs/heads/master
# 8a9ed21916ea47467cd5d221c42c8c2c32ec4cd7
git cat-file -p 8a9ed21916ea47467cd5d221c42c8c2c32ec4cd7
# tree 57f43caf06e8ea6ab879dcaee2e8e1f23da4837d
# parent 4ff7833ba1019717830310d2e7bb9bb43c039e28
# author Chris Couzens <ccouzens@gmail.com> 1519159220 +0000
# committer Chris Couzens <ccouzens@gmail.com> 1519159220 +0000
#
# Add git tree section
cat .git/objects/8a/9ed21916ea47467cd5d221c42c8c2c32ec4cd7 | zlib-flate -uncompress
# commit 237tree 57f43caf06e8ea6ab879dcaee2e8e1f23da4837d
# parent 4ff7833ba1019717830310d2e7bb9bb43c039e28
# author Chris Couzens <ccouzens@gmail.com> 1519159220 +0000
# committer Chris Couzens <ccouzens@gmail.com> 1519159220 +0000
#
# Add git tree section
cat .git/objects/8a/9ed21916ea47467cd5d221c42c8c2c32ec4cd7 | zlib-flate -uncompress | sha1sum
# 8a9ed21916ea47467cd5d221c42c8c2c32ec4cd7  -
```

Just like blobs and trees, we can see that the object starts off with it's type
followed by the length of the content.

Next comes the tree telling git what the commit contains.
This is important, it shows that a commit is stored as a snapshot rather than a
diff.

Then comes the parent commit.
This is the previous commit.

Question: Why might a commit have multiple parents?

Question: Why might a commit have 0 parents?

Finally, it has the author, committer and commit message.
The date it is authored and committed is stored as an ASCII number.
We can see it was stored as a Unix Epoch offset.

```bash
date -d @1519159220
# Tue 20 Feb 20:40:20 GMT 2018
```

Question: What can change a commit's hash value?

Question: Can a commit be its own parent?

Question: Is it possible to get circles in Git commits?

Question: Can you see the connection between how a git commit is stored and
how `git log --graph` displays?

![git tree diagram](https://git-scm.com/book/en/v2/images/data-model-3.png)

Image illustrating commits from the brilliant
[Git book](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects).


## Rebasing

We'll refer back to the commit diagram.

Question: What happens if we rebase to change the message of the 2nd commit?

Question: What happens if we rebase to have ./test.txt contain "version 2" from
the start?

![git rebase diagram](https://git-scm.com/book/en/v2/images/basic-rebase-3.png)

Image illustrating rebasing from the brilliant
[Git book](https://git-scm.com/book/en/v2/Git-Branching-Rebasing).

In the previous diagram we rebase the branch experiment from c2 to c3.

Question: What changes about the commit?
(One thing you know for sure from the diagram.
What else typically changes when you do similar rebases?)

Question: Do old commits disappear when you rebase?

Question: What problems might you get if you rebase a branch that has been
shared?

## Conclusion

We saw that hashes can represent blobs (files), trees (directories) and commits.

We saw how git represents each of these on disk.
We saw that it uses the SHA1 algorithm to generate the Hash key.

We saw that any change you make to a commit or its history changes it into a
different commit.
Changing the contents of a file a few commits previous is enough to change a
commit.

We saw that rebasing changes your history and deviates your history.

For more information, including Ruby examples, check out the excellent Git book.
https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
