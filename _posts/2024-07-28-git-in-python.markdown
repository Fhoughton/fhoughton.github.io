---
layout: post
title:  "Writing A Git Client In Python"
description: Learning how git works by writing a small slice of its functionality in python
date:   2024-07-28 00:19:13 +0100
categories: Git Python
---

I began developing a git client this week, with support finished for the following commands:
- init
- log
- cat-file
- hash-object

In this post I'll detail what I learned and how it related to each command. The repo can be found [here](https://github.com/Fhoughton/PyGit/).

# Where I Learned From
The [official git-scm documentation](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain) was a very useful source of information, with it detailing key information such as the structure of git objects. I also followed parts of [Write Yourself A Git](https://wyag.thb.lt), a guide on writing git implementations, which was particularly useful in helping with code to correctly parse Git 'Objects'.

# Git Init & The Core Of A Git Repo
*Init* creates a new git repo at a specified path. Implenting it teaches us the core of a git repo:
- The .git folder contains all of the actual repository files
- There are three key folders (branches, objects and refs)
- A description file is created but is largely vestigial
- The file HEAD specifies the pathspec of the repo head ('ref: refs/heads/master' by default)
- A config file is contained in every git repo and specifies details such as the git repository format version

![](/images/git-init.png)

# Git Objects & Their Structure
Git stores information about the repository using 'Objects'. There are 4 types of Git Object:
- Blob (raw data such as main.c's contents)
- Commit (a commits information)
- Tag (a named specific commit or object, containing information such as tag date and maker)
- Tree (relates files to folders)

Git objects are compressed using [zlib](https://www.zlib.net/).

When decompressed their format (in order) is:
- A header identifying their type in plain ascii text (blob, commit etc.)
- A space
- The object size in ascii text
- A null character
- The object data (file contents, commit data etc.)

Using this information both *cat-file* and *hash-object* could be implemented (see [here](https://github.com/Fhoughton/PyGit/blob/master/repo.py) for their implementation)

# Git Commits & The Git Log Command
*Log* lists the commit history of the repository, for example when run on the [PyGit](https://github.com/Fhoughton/PyGit/) repository itself:
![](/images/git-log.png)

To implement *log* we first have to parse the object for the specific starting commit. The format is quite complex, but good documentation on it can be found [here](https://wyag.thb.lt/#orgc8b86d2).

Once parsed printing the commit log is as simple as displaying the commit information and then repeating the process until there are no parent commits left:
```python
def print_log(self, hash, seen):
    """Git log display"""
    if hash in seen:
        return
    seen.add(hash)

    commit = self.read_object(hash)
    short_hash = hash[0:8]
    message = commit.commit[None].decode("utf8").strip()
    message = message.replace("\\", "\\\\")
    message = message.replace("\"", "\\\"")

    if "\n" in message: # Keep only the first line
        message = message[:message.index("\n")]

    print(f"{hash}: {message}")
    assert commit.fmt==b'commit'

    if not b'parent' in commit.commit.keys():
        # Base case: the initial commit.
        return

    parents = commit.commit[b'parent']

    if type(parents) != list:
        parents = [ parents ]

    for p in parents:
        p = p.decode("ascii")
        self.print_log(p, seen)
```