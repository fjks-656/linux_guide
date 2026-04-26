# arch安装指南
## 准备工作
### 预留空间
 使用图吧工具箱中的DiskGenius,为你的linux系统留出至少20G空间

### 制作启动盘
   1. 从官网下载iso镜像https://archlinux.org/download/
   2. 将u盘格式换为fat32(windows自带)
   3. 将iso文件解压直接复制到u盘根目录下
   4. 插入 u盘重启

   tips：若重启后未自动进入alive环境，可在启动时狂摁f2（不同电脑可能键位不同）进入BIOS,将启动项中u盘的优先级设为最高，**关掉安全启动**。启动alive环境时由于要将u盘中的系统拷贝到内存，时间较长，请耐心等待


## 安装系统
### 1. 连接网络
```
#进入iwctl
iwctl
#看网卡设备
device list
#一般输出是wlan0，如果不同，记得将后面的wlan0改为你的网卡设备名
#扫描wifi
station wlan0 scan
#列出网络
station wlan0 get-networks
#连接wifi
station wlan0 connect "wifi名称"
#输入密码(无回显是正常现象，输完回车即可)
exit 
#测试
ping -c 3 archlinux.org
```
###  2. 分区
` lsblk`查看磁盘信息

使用图形化的工具分区

```cfdisk /dev/nvme0n1 # nvme0n1为你的磁盘名称```

选择GPT,分两个区
1. efi分区 类型 EFI System，大小512M
2. 根分区 类型 Linux Filesystem， 剩余空间
 
 注：交换分区（虚拟内存）可以之后在根分区创建swap文件

 接下来根式化分区
 ```
# 注意！！！千万看清是那个区，不要将windows的盘给格式化了！！！
# 格式化 EFI 分区
mkfs.fat -F 32 /dev/你的efi分区

# 格式化根分区（推荐使用Btrfs格式）
mkfs.btrfs /dev/你的根分区
 ```
？创建子卷

## 桌面环境


## 桌面应用