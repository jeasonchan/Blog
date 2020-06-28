# 常用操作
更换源
```bash
cd /etc/apt/
sudo cp sources.list sources.list.bak	#备份原文件到统一文件夹下
sudo vim sources.list	#使用vi编辑元列表
sudo apt update	#更新的本地的源列表的软件索引列表
sudo apt upgrade	#更新本机软件，直到版本号和索引中的一致，看自己是否有更新的需求
```
或者
```bash
sudo apt edit-sources #自动询问要使用的编辑器
sudo apt update
```


---
launchpad

---


