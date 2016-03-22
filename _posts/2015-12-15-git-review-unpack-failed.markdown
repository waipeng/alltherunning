---
layout: post
title:  git review unpack failed
categories: notes
---
Recently, I ran into a problem doing a `git review` with just a changed
commit message (no changed files). The error message was like:

    # git review
    error: unpack failed: error Missing tree 6f9d7b5d127584654beeb2cb0f160c21e521cf01
    fatal: Unpack error, check server log
    To ssh://user@gerrit.example.com:29418/internal/repo.git

To solve this, push with the `--no-thin` option

    # git push --no-thin gerrit HEAD:refs/publish/master
    Counting objects: 5, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 3.42 KiB | 0 bytes/s, done.
    Total 5 (delta 0), reused 0 (delta 0)
    remote: Processing changes: updated: 1, refs: 1, done
    remote: (W) e98f57f: no files changed, message updated
