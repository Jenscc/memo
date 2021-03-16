# ARCH 安装避坑(ARCH 入坑指南)

## BIOS

确保是 UEFI,并关闭 secure boot

## 联网

大概是在 20 年 6 月，wifi-menu 被移除了，所以只能用 wpa_supplicant

```shell
ip link #查看无线设备名称，通常为wlan0
ip link set wlan0 up #开启你的无线设备
iwlist wlan0 scan| grep ESSID #扫描
wpa_passphrase wifi名称 wifi密码 > internet.conf
wpa_supplicant -c internet.conf -i wlan0 & #-c 指定配置文件 -i 指定设备
# 等待+回车
dhcpcd &
# 等待+回车
ping baidu.com #如果前面没有error,应该能ping通
```

### **更新系统时间**

使用`timedatectl`命令来确保时间是同步的：

```shell
timedatectl set-ntp true
timedatectl status # 确保设置成功
```

### **磁盘分区**

使用`fdisk`进行磁盘分区：

```shell
fdisk -l # 查看磁盘设备
....
fdisk /dev/nvme0n1 # 对你的磁盘进行分区，下面默认分引导，主硬盘，交换三个分区
```

详细描述见**[安装指南](<https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)**和**[fdisk 官方文档](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php/Fdisk)**

## 格式化分区

```shell
mkfs.fat -F32 /dev/nvme0n1p1 # UEFI引导分区
mkfs.xfs /dev/nvme0n1p2 # 引导和交换分区外的其他分区
mkswap /dev/nvme0n1p3 # 交换分区
swapon /dev/nvme0n1p3 # 开启交换分区
```

## 挂载分区

```shell
monut /dev/nvme0n1p2 /mnt # 首先挂载主分区
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot # 挂载引导分区
```

## 运行安装脚本

```shell
pacstrap /mnt base linux-zen linux-firmware
```

在 live 环境中使用`reflector`进行镜像的管理，貌似你一连接网络，live 系统会自动执行 reflector 命令来帮你选择镜像源，默认的是根据下载速率进行排序，所以我们应该可以跳过修改镜像源，如果有问题，那么

```shell
vim /etc/pacman.d/mirrorlist # 镜像源

# /etc/pacman.d/mirrorlist
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch

vim /etc/resolv.conf # DNS

# /etc/resolv.conf
nameserver 114.114.114.114
```

## 配置系统

### **生成 fstab 文件**

用以下命令生成`fstab`文件，其中`-U`选项用来设置 UUID：

```text
genfstab -U /mnt >> /mnt/etc/fstab
```

然后使用`cat /mnt/etc/fstab`命令检查以下文件是否正确（每个分区占一行）

### **进入到安装的系统**

```text
arch-chroot /mnt
```

### **安装文本编辑器**

现在的新系统连默认的文本编辑器`nano`都没有了，所以需要自己手动安装一个，不然后面的一些配置无法实现，所以我选择最强的`vim`：(建议直接上 neovim)

```shell
pacman -S neovim
ln -s /usr/bin/nvim /usr/bin/vi #软链接，你的vi何必是vi
ln -s /usr/bin/nvim /usr/bin/vim
```

### **时区**

使用下面的命令设置时区：

```text
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

然后使用`hwclock`生成`/etc/adjtime`文件：

```text
hwclock --systohc
```

### **本地化设置**

> 本地化的程序与库若要本地化文本，都依赖**[Locale](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php/Locale)**，后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准等等。在下面两个文件设置：`locale.gen`与`locale.conf`。

1. 首先编辑`/etc/locale.gen`文件，然后将需要的地区的注释移除，建议将`en_US UTF-8`和`zh_CN UTF-8`都取消注释。
2. 执行`locale-gen`命令生成 locale。
3. 创建`/etc/locale.conf`文件并编辑`LANG`这一变量（将系统 locale 设置为`en_US.UTF-8`，系统的`Log`就会用英文显示，这样更容易问题的判断和处理。）：

```text
LANG=en_US.UTF-8
```

_这里最好不要设置为中文 locale，会导致 TTY 乱码_

### **网络设置**

1. 创建`/etc/hostname`文件设置主机名，假设为`myhostname`
2. 配置`/etc/hosts`文件，将以下内容添加进去：

```text
127.0.0.1 localhost
::1 localhost
127.0.1.1 myhostname.localdomain myhostname
```

### **设置 root 密码**

使用`passwd`命令设置 root 密码即可。

### **安装及配置引导程序**

> 安装引导程序之后才能进入系统

我用的引导程序是**[GRUB](https://link.zhihu.com/?target=https%3A//wiki.archlinux.org/index.php/GRUB)**，首先安装必要的软件包：

```shell
pacman -S grub efibootmgr intel-ucode os-prober # AMD请装amd-ucode
```

这里详细介绍一下 UEFI 系统如何安装配置 GRUB：

1. 首先使用以下命令安装到系统：

```shell
mkdir /boot/grub
grub-mkconfig > /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux
```

### **安装 wifi 网络管理工具**

为你的系统安装网络工具

```shell
pacman -S wpa_supplicant dhcpcd
```

#### 注意：进入你自己的系统后，无线设备名称会发生改变

## reboot

1. 输入`exit`或按`Ctrl+d`退出`chroot`环境
2. 用`umount -R /mnt`手动卸载被挂载的分区
3. 执行`reboot`重启系统
4. 拔掉安装盘

## 再次配置

```
login: root #输入root登入root用户
```

参照**[联网]()**，配置好网络连接

```shell
pacman -Syyu
pacman -S base-devel
```

添加个人用户

```sh
user add -m -G wheel your_username
passwd your_username
```

使用`visudo`命令，修改用户组权限，找到`%wheel`取消其注释

`exit`退出 root,登陆你的个人用户

### 添加 archlinuxcn 源

编辑文件/etc/pacman.conf

```shell
sudo vim /etc/pacman.conf
```

在最后一行加入

```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

### 添加 PGP 密钥

```shell
sudo pacman -S archlinuxcn-keyring
```

### 更新系统

```shell
sudo pacman -Syyu
```

## 图形化界面

安装 kde

### **安装显示服务器**

使用下面的命令安装开源的 xorg

```shell
sudo pacman -S xorg xorg-server
```

### 安装显卡驱动

根据自己的显卡配置来选择安装即可。

对于 intel 显卡，我安装的是官方的`xf86-video-intel`驱动：

```shell
sudo pacman -S xf86-video-intel
```

对于 NVIDIA 显卡，安装开源驱动`nouveau`：

```shell
sudo pacman -S mesa xf86-video-nouveau
```

### 安装登录管理器

推荐使用 SDDM

```shell
sudo pacman -S sddm sddm-kcm
systemctl enable sddm
```

### 安装桌面环境

```shell
sudo pacman -S plasma kde-applications
```

### 声音管理器

```shell
sudo pacman -S alsa-utils pulseaudio pulseaudio-alsa
```

### 蓝牙

用不到的话建议不装

```shell
sudo pacman -S bluez bluez-utils bluez-firmware
```

启动蓝牙

```shell
systemctl enable bluetooth
systemctl start bluetooth
```

### 安装 Aur 助手

```shell
sudo pacman -S yay
yay --aururl "https://aur.tuna.tsinghua.edu.cn" --save
```

之后可以用`yay`来替代`sudo pacman`

### 重启

```shell
reboot
```

之后尽情配置美化你的 arch 吧
