# 1 背景

最近在使用一台全新的服务器，需要挂载磁盘，在此记录一下磁盘的挂载过程。

参考文档：
https://blog.csdn.net/ybdesire/article/details/79145180

# 2 步骤

## 2.1 fdisk -l 查看所有的磁盘
```
Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048 960335871 960333824 457.9G 83 Linux
/dev/sda2       960337918 976771071  16433154   7.9G  5 Extended
/dev/sda5       960337920 976771071  16433152   7.9G 82 Linux swap / Solaris

Disk /dev/sdb: 2.7 TiB, 3000592982016 bytes, 5860533168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 1AB82561-C5FD-47A9-9A29-31835617EBD3

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 5860532223 5860530176  2.7T Microsoft basic data
```

从上面的信息可以看，检测到了两块硬盘sda和sdb，sda分成了三个分区sda1、sda2、sda5,sdb则只划分了一个分区sdb1，而其中，sdb正是我们需要挂载的硬盘，而sdb1是我们挂载的分区。


## 2.2 给分区新建一个访问路径
```bash
mkdir   /data
```

## 2.3 查看/data的文件类型
因为新建的/data路径入口点是在sda上的，所以，新的硬盘的文件格式应该和/data所在的分区的文件格式相同

```bash
root@ubuntu:/home/ubuntu# df -T
Filesystem     Type     1K-blocks    Used Available Use% Mounted on
/dev/sda1      ext4     472500496 1888700 446587068   1% /
```
可见，/data是位于/下面的，而/的文件系统是ext4,因此/data的文件系也是ext4

## 2.4 挂在磁盘到访问路径

```bash
root@ubuntu:/home/ubuntu# mount -t ext4 /dev/sdb1 /data1/
```

## 2.5 查看新挂载的磁盘

```bash
root@ubuntu:/home/ubuntu# df -h
/dev/sda1       451G  1.9G  426G   1% /
/dev/sdb1       2.7T   74M  2.7T   1% /data1
```

今后访问/data1，就相当于访问磁盘sdb1

# 3 小结
跟磁盘相关的几个常用命令：

df -h：查看磁盘占用情况

df -T：查看所有磁盘的文件系统类型(type)

fdisk -l：查看所有被系统识别的磁盘

mount -t type device dir：挂载device到dir
