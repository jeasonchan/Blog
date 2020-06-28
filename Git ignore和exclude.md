# 1 背景

在git中如果想忽略掉某个文件， 不让这个文件提交到版本库中，可以使用修改 .gitignore 文件的方法 或者 exclude方法，两种方式都是以文件的形式存储的，都遵循相同的匹配规则，这个文件每一行保存了一个匹配的规则。

gitignore和exclude的区别是，gitignore文件会跟随commit一起提交上去，是项目的使用这共用这份忽略规则，exclude仅对自己本地有效。

# 2 匹配规则

比如：

```
# 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```



```
# Compiled source #
###################
*.com
*.class
*.dll
*.exe
*.o
*.so

# Packages #
############
# it's better to unpack these files and commit the raw source
# git has its own built in compression methods
*.7z
*.dmg
*.gz
*.iso
*.jar
*.rar
*.tar
*.zip

# Logs and databases #
######################
*.log
*.sql
*.sqlite

# OS generated files #
######################
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Icon?
ehthumbs.db
Thumbs.db
```

# 3 补充

.gitignore 还有个有意思的小功能， 一个空的 .gitignore 文件 可以当作是一个 placeholder，因为正常情况下，空的文件夹是不能commit的 。当我们需要为项目创建一个空的 log  目录时，可以在 log 目录 在里面放置一个空的 .gitignore 文件。这样当我们 clone 这个 repo  的时候 git 会自动的创建好一个空的 log 目录了。

