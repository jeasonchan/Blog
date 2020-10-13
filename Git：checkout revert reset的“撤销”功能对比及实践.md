# 1背景
自己写小练习时，写回退到xxxx commit ID，然后自然的使用了get revert xxxx的操作，最后提示路径问题，让我解决冲突……很奇怪，为毛有路径问题，难道revert功能用错了？？于是就百度了一下，git的撤销功能大全：checkout、revert、reset。
先小结一下：
revert，撤销，可撤销某单次的commit、某单次commit中的某个文件修改，当然，后面基于该commit的修改，自己要去解决冲突……
checkout，切换，可切换至某个commit，切换的commit之后的东西当然暂时看不见了，可来回反复切换，向合并进主分支时，再次commit即可。
reset，之前有用过，有mixed、hard、soft三种程度的reset
# 2 checkout
# 3 revert
# 4 reset
# 5 参考
<https://www.jianshu.com/p/cc87d5314c80>
