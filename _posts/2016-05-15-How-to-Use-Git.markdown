---
layout:     post
title:      "Git常用命令"
date:       2016-05-15
author:     "Sim"
catalog: false
tags:
    - Git
---

1. 设定

安装完成以后，使用终端输入`git --version`,如果安装成功的话，就会输出git的版本号。

然后可以设置git的名称和Email

```
git config --global user.name "<Your Name>"
git config --global user.email "<youremail@example.com">
```

2. 建立文件夹`$ mkdir MyDoc` -> 进入文件夹`$ cd MyDoc` -> 初始化git`$ git init`

使用`git status`来确认是否初始化成功。一般情况下都可以使用此命令来查看仓库当前的状态。

3. 添加新文件时使用`$ git add "DocName"`来添加

4. 提交修改使用commit： `$ git commit -m "<your commit message>"`

5. 使用`$ git diff`来查看不同。

6. 'readme' 通常是用来解释一个程序的功能、使用方法等

7. '.gitignore' 是要忽略的档案清单，通常是告诉git在版本控制记录的时候不要理会这些文件

8. 'License'是用来声明程序的使用。在[choosealicense.com](choosealicense.com)可以找到参考

9. 连接远端和本地`$ git remote add origin <URLFORMGITHUB>`

10. 推送到远程仓库: `git push origin master`

11. 下载其他项目的副本: `git clone <URLFORMGITHUB>`

12. 连接到原始的Repository： `git remote add upstream <URL>`.其中'upstream'为连接命名

13. 新增分支：`git branch <BRANCHNAME>`

14. 取出分支：`git checkour <BRANCHNAME>`

15. 拉取远程修改到本地：`git pull <REMOTENAME> <BRANCHNAME>`

16. 检查远程仓库是否有变动： `$ git fetch --dry-run`

17. 合并：

  1) 切换到想合并进去的分支: `git checkout gh-pages`

  2) 合并哪个分支: `git merge <BRANCHNAME>`

  3) 删除已合并的分支: `git branch -d <BRANCHNAME>`

  4) 从原程序中pull：`git pull upstream gh-pages`
  
---

# 参考链接

1.[Git-it](http://jlord.us/git-it/index-zhtw.html)
