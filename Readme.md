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

The question I hope to answer today is where these commit hashes come from.

## Git Plumbing

> Git is fundamentally a content-addressable filesystem with a VCS user interface written on top of it

From the [Git Book](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain).
