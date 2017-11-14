> 详细参见
http://vbird.dic.ksu.edu.tw/linux_basic/0420quota_3.php

- 如果磁盘大于4T ,需要先做如下操作,
```
parted /dev/dm0
(parted) mklabel gpt
(parted) unit TB
(parted) mkpart primary 0.00TB 4.00TB
(parted) print
(parted) quit
mkfs.ext4 /dev/dm0p1
```
- 常用操作

>- 1.直接使用新硬盘创建LVM   
-  2.扩展现有LV


- 1，直接使用新硬盘创建LVM

```
a.fdisk /dev/sdb (创建分区槽)
n->p->1->两个回车->t->8e->w
刷新磁盘 partx -a /dev/sdb
b.pvcreate /dev/sdb1 /dev/sd*
c.vgcreate -s 16M {VGNAME} /dev/sdb1 /dev/sd*
d.
使用全部
lvcreate -n lvcrawler -l 100%FREE {VGNAME}
指定大小
lvcreate -L 1279 -n {LVNAME} {VGNAME}
lvdisplay查看是否分配成功
e.mkfs -t ext4 /dev/{VGNAME} /{LVNAME}
f.mout 挂载
在/etc/fstab 里面添加
UUID=.. "{挂载目录}"   ext4 defaults 0 0
mount -a 将盘挂载好。

为什么使用UUID：
UUID是磁盘唯一标示，而/dev/sbd1是系统检测磁盘时按顺序给的名字，可能导致两次重启磁盘名字对应不同的真实磁盘
查看磁盘UUID
blkid /dev/sda
```

- 2.扩展现有LV

```
a.fdisk /dev/sdb 将新硬盘创建为分割槽备用
b.pvcreate /dev/sdb4
c.vgextend {VGNAME} /dev/sdb4

使用vgdisplay查看未分配 的PE 数量
--- Volume group ---
VG Name vg_opt
System ID
Format lvm2
Metadata Areas 2
Metadata Sequence No 3
VG Access read/write
VG Status resizable
MAX LV 0
Cur LV 1
Open LV 1
Max PV 0
Cur PV 2
Act PV 2
VG Size 179.99 GiB
PE Size 4.00 MiB
Total PE 46078
Alloc PE / Size 5118 / 19.99 GiB
Free PE / Size 40960 / 160.00 GiB <--新硬盘剩余PE数目
VG UUID SIkAff-SUWb-yn8r-PCUz-wA6f-SWxt-YBjpbf

d.lvextend --size +40G {lvpath}
###lvresize -l +nnn(新硬盘剩余PE数目) {lvpath} :当需要减少磁盘空间，可以供lvresize,增加最好用lvextend,减少不必要的风险，比如resize误操作到比之前的空间还小。
e.
- 如果是ext2, ext3 or ext4 :  
resize2fs {lvpath} 将LV扩充，应用到filesystem. (此操作可能有数据丢失风险，在对数据一致性要求比较高的情况下，最好先umount再mount)
- 如果是xfs:  
xfs_growfs {lvpath}

```
