---
layout:     post
title:      "iTerm2免密码登录远程服务器"
date:       2016-08-03
author:     "Sim"
catalog: false
tags:
    - iTerm2
---

老早以前就搞了个iTerm2装在机子里，记得刚装那会折腾了好久都没办法免密码登录服务器，后来就放弃了。导致现在每次登录远程服务器或者上传文件都要敲一堆密码才能登录。实在是烦得不行。晚上bing找了一下，就搞定了。。。

写个脚本内容如下

```
#!/usr/bin/expect -f
set user username
set host ip
set password password
set timeout -1

spawn ssh $user@$host
expect "*assword:*"
send "$password:\r"
interact
expect eof
```

然后随便保存，我是将其保存在~/.ssh下面

打开iTerm，ctrl+O打开配置，添加profile，选择Command-Command,写入'expect ~/.ssh/your_shell。保存后执行就可以了。



