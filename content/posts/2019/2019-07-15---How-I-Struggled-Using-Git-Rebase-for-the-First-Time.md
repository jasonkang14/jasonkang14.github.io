---
title: Git Rebase - How I struggled using it for the first time
date: "2019-07-15T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/git-rebase"
category: "GIT"
tags:
  - "Git"
description: "How I overcame my confusion using git rebase"
---

I was taught to use `git rebase` from now on before making a `pull request`. Honestly, the command it self was confusing. This is how I understood `git rebase`

1. You are `rebasing` the `master` branch.
   The command for `git rebase` is `git rebase -i master`. `-i` represents interactive, which allows you to choose commits to include as you rebase your master branch.

After using the command, you face a message like the image below;
![git rebase](https://scontent-gmp1-1.xx.fbcdn.net/v/t1.0-9/67501828_10219354644757759_6212543380459618304_o.jpg?_nc_cat=110&_nc_oc=AQkU3NMI1N-Dt3nhhMx3X3hjcBTPU36dROO_QyGiD2i3tsOx4NjDBAiA8LESjLeZ-Lo&_nc_ht=scontent-gmp1-1.xx&oh=b3b11394c762f66891a3e793624a49ab&oe=5DC07CD2)

Using the commands explained, you can choose which commits to use. If you are going to use all the commands, you just have to leave the top commit as pick, and choose the rest as s, or `squash` so that all the commits can be `squashed` together.

2. Conflicts may occur in every step.
   Forgot to take a screenshot of it, but I faced 3 conflicts while rebasing 7 different commits. You have to be careful as you resolve the conflicts.

So then, why do you have to use `git rebase` when you can simply just `git add` and then `git commit`?

`git rebase` ensures that the orders of commits do not get messed up. When you make a `pull request` the master/develop branch pulls each commit based on the time that it was made. Therefore, it is difficult to track down the history of each commit. And it gets worse if you have to revert your commit since commits from different feature branches are all mixed together.

By using `git rebase`, you need to take more time before making a `pull request`, but you can prevent the master branch from having mixed-up history of commits from different branches.
