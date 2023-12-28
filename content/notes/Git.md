---
title: Git
date: 2023-12-18
tags:
  - seed
enableToc: true
---
# Notable Git notes

Sources:
- https://learngitbranching.js.org/
- https://cs.fyi/guide/git-cheatsheet
- https://www.youtube.com/watch?v=Sqsz1-o7nXk&ab_channel=TheModernCoder

### Merging vs. Rebasing
- git rebase solves the same problem as git merge
- merging is nice because it is a non-destructive operation in that the existing branches do not change in any way
- rebasing re-writes the project history by creating brand new commits for each commit in the original branch. The major benefit here is that you get a much cleaner project history
- NEVER run git rebase on public branches 