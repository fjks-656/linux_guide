# 扩展
__该文档描述了我解决一些问题的方案__
## grub配置
*请注意更改下列配置项后均需要运行下面命令来更新grubconfig*

`sudo grub-mkconfig -o /boot/grub/grub.cfg`

### 图形化设置界面

如果不想手动更改配置文件，可以选择使用grub-customizer，但这会导致权限问题，wayland环境不允许root下运行图形化界面，需要以下配置

`
sudo pacman -S grub-customizer

sudo pacman -S xorg-xhost

xhost +local: root

pkexec grub-customizer
`

接下来即可隐藏系统或更改名称

### 隐藏“高级设置 (Advanced options)”
在文件/etc/default/grub中找到以下一行并取消注释：

`#GRUB_DISABLE_SUBMENU=y`

### 隐藏“UEFI 固件设置”
运行命令

`sudo chmod -x /etc/grub.d/30_uefi-firmware`

### 设置主题
解压项目中的grub_theme.zip文件，使用其中的install.sh脚本安装主题。

如果没有图形界面，可以查看grub配置文件/etc/default/grub，中
找到GRUB_TERMINAL_OUTPUT一行，取消注释，改为  `GRUB_TERMINAL_OUTPUT="gfxterm" `

### 未解决的问题

目前发现我的efi盘挂载到了/boot/efi下，导致/boot实际是在我的系统盘，是btfs格式，会产生松散文件，grub不支持，导致其无法保存上一次的启动项

## rime输入法

### 开启后是繁体中文
摁下`Ctrl+~`或`f4`即可设置

### 配置垂直显示(fcitx)
运行`fcitx5-config-qt`选择附加组件，选第一个经典用户界面，里面有垂直候选界面，点击勾选

### 提交预览
如果你不希望在打字是拼音字母直接出现在输入框中，可以点击中州韵的配置，将预编辑模式改为不显示

### 输入法皮肤
可在github上自行搜索fcitx5-themes，然后按星排序

本项目中提供了仿macOS Sonoma Dark主题，如果要应用它，请先解压，然后将其复制到`~/.local/share/fcitx5/themes`目录下（如果没有该目录，请创建一个）
