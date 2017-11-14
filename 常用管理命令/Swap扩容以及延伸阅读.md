swap并非可有可无，本文先从日常扩容讲起，最后通过两种使用场景分析swap的使用策略。

---
### 扩容
原理:先使用swapoff将swap关闭，此时swap中的内容会还原到内存中，配置完之后使用swapon再重新启用swapon.  
所以内存剩余量必须大于swap交换量。

扩容方式：  
- 新建swap分区
- 新建swap文件
- 扩展现有的swap lvm2分区  

此处采用扩展swap lvm2分区，步骤如下：  
假设swap分区为/dev/VolGroup00/LogVol01  
```
- 1.先禁用swap分区  
swapoff -v /dev/VolGroup00/LogVol01
- 2.扩展swap分区  
lvm lvresize /dev/VolGroup00/LogVol01 -L +256M
- 3.格式化swap分区  
mkswap /dev/VolGroup00/LogVol01
- 4.激活swap分区  
swapon -va
- 查看swap分区信息  
cat /proc/swaps 或者 free
```

>因为并未找到swapoff完全安全的证据，所以最好制定maintenance window来扩容。

---
### swap是否可有可无？  
#### 分两种使用场景进行分析
- 1.在无状态且有高可用支撑的系统如web前端，可以关闭。  
因为即使内存OOM也可以被入口健康监测踢除，而不会影响服务；反之，使用swap，可能导致请求被转发到由于内存不存而性能受影响机器上。
- 2.其他情况，可以开启。  
并配置合理的swappiness,这样可以降低内存突增带来的OOM风险，同时要加强swap空间使用监控，及时处理内存不存以及使用swap不释放的情况（如某些JAVA进程）。
#### 使用swappiness配置swap策略  
  内核参数swappiness控制使用swap空间的等级，越低内核会尽量少使用swap，反之亦然。  
  ```
  vm.swappiness=10
  ```

---
### 延伸阅读
- 什么是swap
  - swap用于存放内存转存出来的的非活动内存页。
  - 延伸:什么是内存页？  
简单来说，内存页为虚拟内存寻址的最小单元，其保存真实的物理内存地址，此处的swap换出，即将非活动内存页对应的物理内存转存到硬盘。（另建文章详解内存页）
- 如何配置swap分区  
小内存可以配置1到2倍空间，大内存（>64G）可配置12-16G,如果启用休眠需要更大。  
实际并没有标准参考，以上仅为一个范围，可根据实际情况具体调整。
- 谁用了swap?  
free/top可以查看目前系统swap总体用量，可以通过如下方式查看具体谁用了swap空间。
> 由于共享内存的存在，进程的vmswap数值可能包含一个或多个共享内存，该共享swap在数值上被多个进程计算，但实际只使用一份swap空间，所以无法准确的计算出每个进程占用的实际swap空间，不过实际应用中，只需要定性即可。

  - 使用/proc/*/status
  ```
  for file in /proc/*/status ; do awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 2 -n -r | less
  ```
  按swap使用量列出进程。
  - smem  
  可以按用户，按进程等列出swap空间，如：
  ```
  smem -u
User     Count     Swap      USS      PSS      RSS
rpcuser      1     3568        4       41      668
vivek        4     7300       44       73      564
xxxxxxxx     3     6120       56       77      524
rpc          1      200       68      104      596
raj          1      468      272      300      892
ntp          1      316      324      367     1036
cdnnginx     1      420      572      603     1216
#USS-非共享空间
#PSS-非共享空间+共享空间
```
---
>参考  
http://t.cn/R9QVjxM
