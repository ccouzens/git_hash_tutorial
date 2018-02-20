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
