---
layout: post
title:  git基础命令记录
category: base-command
tags: [base-command]
description: git基础命令记录
---

### .gitignore不起作用
```xml
    git rm -r --cached .
    git add .
    git commit -m 'update .gitignore'
```

### tag
#### tag创建
- git tag v1.0
- git tag -a v0.1 -m "version 0.1 released"

### tag提交

- git push origin v1.0   提交具体的tag
- git push origin --tags   提交所有的tag
    
### 删除远程tag
```xml
git push origin --delete tag <tagname>
```

### 撤销指令
当我们编写的内容发现有问题，这时我们想回到原来的内容，就可以使用此命令撤回。
```xml
git checkout <file>
```
注意事项：指令该是先从缓存区中拉取版本还原，如果没有再到版本库中拉取还原。
缓存区：git add 
版本库：git commit 