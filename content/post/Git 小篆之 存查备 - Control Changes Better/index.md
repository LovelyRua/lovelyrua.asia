---
title: Git 小篆之 存查备 - Control Changes Better
description: Git 日常操作实用指南：告别手忙脚乱，掌控你的代码版本
date: 2025-05-06 09:32:23+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Git
    - Essay
---

git status  // 让我看看怎么个事

git restore <file>  // 按下回车就和你的文件撒哟娜拉了

*git checkout -- <file>  // 撒哟娜拉 in 2000s

git add <file>  // 温柔的把写完的胡话扔到篮子里

git restore --staged <file> // 撤销暂存区修改 add→不add

*git reset HEAD <file>  // 旧撤销暂存区修改 不 restore

git rm --cached <file>  // 暂存区丢弃 保留工作区文件 add 了忘加 .gitignore 的文件用

---

git commit -m <info>  // 把胡话 push 就没脸见同事了

git log  // 查有没有提交胡话

git reset --soft HEAD^  // 还好没 push 消灭证据把胡话改了

git reset --hard HEAD~1 // 时光机

git reflog  // 哎我操胡话和代码全滚了

---

git switch <branch> // Nintendo

*git checkout <branch>  // HELLO WORLD

git branch -m <old> <new>  // 农奴翻身做主人

git merge --no-ff -m <commit> <branch>  // 合并胡话到当前分支

git merge --squash <branch>  // 打包合并

git cherry-pick <commit>  // 重放提交到当前分支

git rebase <branch> // 将当前分支的起点移到分支的最新处

---

git clean -fd  // node_modules backhole.gif

---

重磅消息：为了解决和文件撒哟娜拉的困扰 我们引入了胡话烘干桶！

git stash  // 写一半胡话去玩任天堂

git stash list  // 列出烘干胡话

git stash apply stash@{0}  // 把胡话干夹在胡话里

git stash pop stash@{0}  // 吃掉胡话干

git stash drop stash@{0}  // 扔掉过期胡话干

git restore --source=stash@{0} -- <file>  // 胡话干占领地球

*git checkout stash@{0} -- <file>  // 旧胡话干占领地球


> * tagged is gitting old