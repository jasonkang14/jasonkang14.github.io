---
title: Git - allow-unrelated-histories
date: "2020-05-14T21:53:37.121Z"
template: "post"
draft: false
slug: "/git/allow-unrelated-histories"
category: "Git"
tags:
  - "Git"

description: "How to fix an error when you try to merge unrelated branches"
---

To be honest with you, you should never face such problem, but I did. When I try to make my very first pull request to the develop branch, I faced an error saying, `refusing to merge unrelated histories`

But honestly, how are they do they not share any commit histories in common? They share the same repository for sure, when whenever I push my work to the remote branch, it has been working completely fine. But when I tried to make a PR to the develop branch, I failed due to `unrelated histories`.

The problem was somewhat easy to solve for me in this case since all the works had been done in the feature branch. So honestly all that the develop branch had was its initial settings, so I could easily overwrite the develop branch using the `--allow-unrelated-histories` option.

It literally allows you to merge unrelated histories. I was able to merge the develop branch to the feature branch and then successfully make a PR. Another option could be a `rebase`, which I did not try because I was not ready to rebase over 500 commits before making a PR. Hopefully, you will manage your work well so that you won't have to face such problem.
