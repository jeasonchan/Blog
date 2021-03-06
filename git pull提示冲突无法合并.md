﻿# 1 问题
git pull拉取最新的时候提示：Please commit your changes or stash them before you merge
# 2 原因分析
自己对代码进行了修改，但是还没有提交，而远端仓库的最新代码同样对自己修改的地方做了修改，git无法判断，以自己的修改还是以远端仓库的代码为合并的版本，于是抛出这个问题。
# 3 解决方法
## 3.1 git reset --hard origin/master（不推荐）
```bash
git reset --hard origin/master  #以远端仓库为准，放弃本地自己的修改
git pull #再拉取一下，确保拉取最新的
```
不想放弃修改的，又想用这个方法的，可以根据之前的提示，备份一下冲突的文件，再自己人工对比 = =……
## 3.2 git stash（推荐）
```bash
#假设之前已经报错了
git stash #将自己的内容暂存到工作栈中
git stash list #这一步可有可无，列出工作栈中所有暂存的内容
git pull #拉取远端仓库内容
git stash pop #从工作站栈取最新的暂存内容，并尝试合并到pull下来的版本中
```
这时候控制台提示，合并冲突，让你自己决定取舍。这时候，代码中会出现这样的片段：
```java
<<<<<< updated upsterams
	(远端仓库的代码)
======
	(从stash的工作栈取出来的，和远端仓库冲突的代码)
>>>>>>
```
自己选择删除一段即可，记得删除代码和git生成的分隔符什么的，要是冲突的地方太多，可以将代码文件以vscode打开，会自动提示冲突的地方，也能一键式保留自己需要的版本，不要自己手动删除多行东西。选定保留的代码后，再执行以下操作：
```bash
git pull #再确认一下下拉的为最新的
git stash clear #清除自己暂存在工作栈的所有内容，很多教程都没这一步，结果暂存工作栈越来越大，无语……
```
