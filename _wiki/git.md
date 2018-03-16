---
layout: wiki
title: Git
categories: [git]
description: git的日常
---

### 删除某个文件的历史

> 不小心上传了很多乱七八糟的文件，rm后再次提交，但是`.git`里还是有记录，导致库很大怎么办？

	git filter-branch -f --tree-filter 'rm -rf rails/nocturne/app/test' HEAD

	git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin

	git reflog expire --expire=now --all

	git gc --prune=now

	git push origin --force

### 回退commit

	git reset --hard <hashcode>

	git push origin master -f

### 切换远程分支到本地

	git checkout -b xxx origin/sss

### 删除本地分支

	git branch -d xxxx

### 删除远程分支

	git push origin :xxxx

### clone指定分支的指定深度

	git clone -b branch url --depth=1

### 下载github某个文件或文件夹

	例如：https://github.com/maotongxue/elk/tree/master/some_tried/zabbix_log_conf

	可以用svn：svn checkout https://github.com/maotongxue/elk/trunk/some_tried

注意把`tree/master`改为`trunk`
