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
### 确认开启时间同步
使用`timedatectl`查看是否开启时间同步
若未开启，请执行`timedatectl set-ntp true`

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
创建子卷
``` mount -t btrfs /dev/你的根分区 /mnt # 将根分区挂载到/mnt
      # 创建两个子卷
      btrfs subvolume create /mnt/@
      btrfs subvolume create /mnt/@home
      umount /mnt  #取消挂载
      # 重新挂载两个子卷
      mount -t btrfs -o subvol=@,compress=zstd /dev/你的根分区 /mnt
      mount --mkd -o subvol=@home,compress=zstd /dev/你的根分区 /mnt/home
      # 挂载esp
      mount /dev/你的efi分区 /mnt/efi
```
保存挂载表
```genfstab -U /mnt > /mnt/etc/fstab```
### 3.使用国内镜像源
运行一下命令，筛选源
``` reflector -a 12 -c cn -f 10 --sort score --v --save /etc/pacman.d/mirrorlist```
运行命令更新密钥
```pacman -Sy archlinux-keyring```
### 4. 安装系统
``` 
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs 
```
 等待一段时间，接下来安装一些基本工具
 ```
passtrap /mnt networkmanager sudo nano vim inter-ucode
 ```
### 5.进入临时系统
``` arch-chroot /mnt```
接下来进行时区，本地化，主机名，密码配置
```
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #设置时区
nano /etc/locale.gen #本地化配置
```
打开nano后搜索`en_US.UTF-8 UTF-8` `zh_CN.UTF-8 UTF-8`并取消注释
运行` locale-gen`生成本地化文件
接着` nano /etc/locale.conf`
输入以下内容将全局默认语言设置为英文
```
LANG=en_US.UTF-8
```
`nano /etc/hostname`后输入你的主机名
`passwd`修改root密码（无回显）


接下来安装grub
```
pacman -S grub efibootmgr # 安装grub及efi引导管理
grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/efi --bootloader-id=arch # bootloader-id后可以自己取一个启动项名字
ln -s /efi/grub /boot/grub #创建软链接，兼容其他软件
grub-mkconfig -o /boot/grub/grub.cfg #创建grub配置文件
```

注意！！！ 确保你的grub是安装到/efi的不然根分区的btrfs格式将会导致你的grub无法写入文件


接下来配置双系统（windows）
```
pacman -S os-prober exfat-utils
os-prober #若能检测到windows说明成功
```
`nano /etc/default/grub`编辑grub配置文件找到`GRUB_DISABLE_OS_PROBER=false`并取消注释


接着将`GRUB_SAVEDEFAULT=true`取消注释以记忆上一次启动项
将`GRUB_DEFAULT`改为`saved`


将`GRUB_CMDLINE_LINUX_DEFAULT`改为`loglevel=5`开启日志


再次`grub-mkconfig -o /boot/grub/grub.cfg `生成grub配置文件
 