# 扩展
__该文档描述了我解决一些问题的方案__
## grub配置
*请注意更改下列配置项后均需要运行下面命令来更新grubconfig*

`sudo grub-mkconfig -o /boot/grub/grub.cfg`

### 隐藏“高级设置 (Advanced options)”
在文件/etc/default/grub中找到以下一行并取消注释：

`#GRUB_DISABLE_SUBMENU=y`

### 隐藏“UEFI 固件设置”
运行命令

`sudo chmod -x /etc/grub.d/30_uefi-firmware`

