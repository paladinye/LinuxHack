#### 物理机
#### 虚拟机
扩容之后如何识别？
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Online_Storage_Reconfiguration_Guide/adding_storage-device-or-path.html
echo "- - -" > /sys/class/scsi_host/host#/scan  
host#, # 是数字，具体参看操作系统可用，数字的含义？在vm中添加的时候，是否决定该数字

fdisk如何判断磁盘是否已经格式化
