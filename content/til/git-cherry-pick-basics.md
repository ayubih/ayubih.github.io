---
date: 2024-01-19T12:00:00-06:00
draft: false
title: 'Git Cherry Pick Basics'
tags: ["til", "learning", "git", "version-control"]
categories: ["TIL"]
description: "How to use git cherry-pick to apply specific commits from one branch to another"
---

## What I Learned

Git cherry-pick allows you to apply specific commits from one branch to another without merging the entire branch.

## Details

The basic syntax is:
```bash
git cherry-pick <commit-hash>
```

This is super useful when you need just one or two commits from a feature branch, or when you need to apply a hotfix to multiple branches.

You can also cherry-pick multiple commits:
```bash
git cherry-pick commit1 commit2 commit3
```

Or a range of commits:
```bash
git cherry-pick start-commit..end-commit
```

## Source/Reference

Learned this while working on a project where I needed to apply a specific bug fix to multiple release branches without merging all the new features.
