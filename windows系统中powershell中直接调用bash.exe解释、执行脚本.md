@[TOC]
# 1背景
在windows上开发，都会装一个git实现，最常用的就是<https://git-scm.com/download/win>这个git了，安装之后，最常用的使用场景是，shift+右键，选择git-bash here，就可以进行git版本管理相关操作了。除此之外，很多人会像我一样，win和linux都会进行一些开发、调试工作，还不错的bash终端就是这个git了，可以直接在git的终端中运行bash脚本，从而启动一些跨平台的程序。
因此，git for windows真是小巧又方便！
在实际使用中，经常打开powershell，因为，powershell中路径和文件名补全很只能，速度很卡，而git终端经常卡死且大小写敏感，而且，powershell中可以直接执git版本库管理的命令，但是，有时候还有执行bash脚本的需求，该如何让powershell识别git中自带的bash解释器呢？？
# 2为什么可以识别git
因为 windows中系统换变量PATH（win对大小写不敏感……）中有这个值**C:\CRsoftwares\Git\cmd**，该路径下有git.exe，在powershell中执行git命令本质就是启动git.exe，并向这个程序传递字符串参数罢了。
# 3将bash.exe添加到PATH中
那肯定还还有一个bash.exe在git安装目录中，随手一搜，果然有，**C:\CRsoftwares\Git\bin**，于是将这个值添加到PATH，新建一个powershell窗口，输入bash，可以识别bash了，完成！
同时，观察该目录发现**C:\CRsoftwares\Git\bin**中也有一个git.exe，将cmd和bin下面的两个git.exe进行对比，发现一模一样，因此，添加该环境变量值并不会影响原先的git功能。
# 4效果
 可以只打开powershell一个终端程序，然后在powershell中执行git、bash，同时又能充分利用powershell的补全功能。
 最重要的是，直接输入bash，皆可以切换到bash终端环境，exit又回到powershell，切换自如！
# 5感悟
1. 环境变量真是到处都有
2. 一些皆文件
