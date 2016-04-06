#ArchLinux安装过程详解
![Arch](<https://raw.githubusercontent.com/KevinChans/java-blogs/master/linux/arch_icon.jpg>)
###一、简介：
Linux有很多个发行版本，有免费的也有商业版本的，每个人可以根据自己的需求不同选择不同的发行版本。Debain和RedHat商业版本稳定性非常好，适合用于服务器，Fedora是RedHat新技术的试验场，适合桌面使用稳定性不是很好，bug比较多；而Ubuntu安装简单上手较快，适合新手，ubuntu在服务器、云计算，甚至是移动端都有广泛的应用；如果想深入的了解整个系统的安装过程，包括磁盘分区、设置语言、网络、字体、时区、图形化界面的安装，推荐使用ArchLinux和Gentoo，折腾之中会有不小的收获，通过安装和使用Linux你能够对各种配置文件有一定的了解。这篇文章写在我的第三次安装Arch之前，写出来分享给大家，话不多说开始！    
###二、安装VirtualBox
我是在Mac上安装Arch，没有选择安装双系统，而是在VirtualBox上安装Arch。
首先需要下载VirtualBox，[点此下载VirtualBox](https://www.virtualbox.org/wiki/Downloads)
安装完VirtualBox之后推荐把对应版本的Extension Pack也安装上，因为Arch的安装过程可能会用到扩展包。
####1.下载系统镜像
arch系统镜像下载地址：<https://www.archlinux.org/download/>，下边可以选择所在地区，选择中国即可，对应的mirror有163和bjtu.edu.cn等，点击一个进入就可以下载了。
####2.创建Virtual Machine
设置虚拟机名字为ArchLinux，系统为Arch64位，内存2048M，创建虚拟硬盘，大小给了100G，点击finish创建完毕。
在虚拟机列表中找到刚刚创建的Arch，然后右键settings，切换到Storage下，在Controller IDE右边点击第一个加号选择下载好的ISO文件，点击OK。然后在虚拟机列表下双击ArchLinux运行就可以开始安装了。Arch启动之后选择第一项Boot Arch Linux (x86-64)来引导安装Arch。
###三、制作启动盘
如果你打算在笔记本或者台式机上直接安装ArchLinux的话，可以使用制作启动盘来引导安装Arch。在此之前我制作启动盘都是通过Ultraiso，在Windows上也是十分方便，基本上10分钟就可以搞定。但是今天我尝试制作Arch启动盘没有成功，所以发现在Linux下有个神奇的命令dd，基本上敲几个命令就好了，代码如下：

```
~ diskutil list #找到需要制作启动盘的disk，名字形如/dev/diskN
~ diskutil umountDisk /dev/diskN #取消挂载
~ sudo dd if=/Users/luffy/Desktop/archlinux-2016.04.01-dual.iso of=/dev/disk2 bs=1m   #写入U盘
```
###四、安装ArchLinux System
####1.磁盘分区
Arch启动之后，默认艺root用户登录，首先要对磁盘进行分区：

```
~ lsblk #查看block设备
```
输入以上命令应该能看到之前我们创建的8g的磁盘，现在我们要分别为/、/home、/boot以及swap进行分区。
使用parted工具对sda进行分区

```
~ parted /dev/sda
```
创建分区表

```
~ mklablel msdos 
```
格式化磁盘为ext4格式,分出boot,swap, /home和/

```
~ mkpart primary ext4 1m 100M
~ set 1 boot on
~ mkpart primary ext4 100m 50g
~ mkpart primary linux-swap 50g 54g
~ mkpart primary ext4 54g 100%
```
格式化并启用swap

```
~ mkswap /dev/sda3
~ swapon /dev/sda3 
```
挂载/(root)分区

```
~ mkfs.ext4 /dev/sda2
~ mount /dev/sda2 /mnt
```
创建/mnt/home文件夹,并挂载home分区

```
~ mkfs.ext4 /dev/sda4
~ mkdir /mnt/home
~ mkfs.ext4 /dev/sda4
~ mount /dev/sda4 /mnt/home
```
创建/mnt/boot文件夹，并挂载boot分区

```
~ mkdir -p /mnt/boot
~ mkfs.ext4 /dev/sda1
~ mount /dev/sda1 /mnt/boot
```
以上，就完成了对磁盘的分区。
####2.设置网络
对于虚拟机上的arch只需要设置如下，就可以访问网络：

```
~ vi /etc/rc.conf #添加行：interface = eth0
~ dhcpcd
~ ping www.baidu.com
```
如果你是真机安装Arch，下面介绍如何设置静态IP地址、网关、DNS，新版本的ArchLInux通过/etc/dhcpcd.conf来配置网络,编辑该文件，设置如下：

```
~ ifconfig    #网卡的名字，比如eth0
~ vim /etc/dhcpcd.conf
interface eth0    #eth0代表网卡名字
static ip_address=10.1.193.15/23    #静态ip地址，23代表子网掩码
static routers=10.1.193.254    #设置网关
static domain_name_servers=10.11.50.65 8.8.8.8    #设置DNS，多个dns用空格隔开
~ dhcpcd -k
~ dhcpcd
```
ok，ping www.baidu.com就会发现可以上网了哦
详细请看ArchWiki：[ArchLinux Dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd)
####3.安装基本软件包
arch安装软件通过pacman,我们首先需要设置一下镜像的地址,之后软件的更新下载都会通过这里面配置的地址去获取，编辑/etc/pacman.d/mirrorlist    

```
~ vi /etc/pacman.d/mirrorlist
```
镜像地址可以通过[mirror](https://www.archlinux.org/mirrorlist/)查看，这里设置为China的镜像地址，如下(一般设置一个就够了)：

```js
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirror.bjtu.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.hust.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.hustunique.com/archlinux/$repo/os/$arch
Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.opencas.cn/archlinux/$repo/os/$arch
Server = http://run.hit.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
```

安装基本的软件包：

```
~ pacstrap -i /mnt base base-devel #使用-i参数，安装前会提示
```
####4.生成fstab

```
~ genfstab -U -p /mnt >> /mnt/etc/fstab
```
将配置文件复制到/mnt的新系统，然后chroot到新系统

```
~ arch-chroot /mnt /bin/bash
```
####5.设置语言、时区、password
编辑/etc/locale.gen，找到以下几行去掉前面的＃
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
然后执行 ~locale.gen ，使更改生效.

```
~ sudo tee /etc/locale.conf <<< LANG=en_US.UTF-8
```
更改时区：

```
~ tzselect    #根据提示选择对应的时区
```
设置root密码

```
~ passwd
```
####6.安装bootloader
这里安装grub

```
~ pacman -S grub os-prober
~ grub-install /dev/sda
~ grub-mkconfig -o /boot/grub/grub.cfg
```
设置主机名

```
~ echo arch > /etc/hostname
~ vi /etc/hosts #编辑该文件并设置为同样的主机名
```
ok,经过以上的配置，Arch已经安装成功了，此时reboot系统，power off之后，从虚拟机的settings中删除掉之前添加的iso，然后重新双击启动arch即可。
####7.创建用户并设置密码
使用root用户登录之后，需要创建用户，使用useradd命令，并设置密码

```
~ useradd archie
~ passwd archie
```
###五、安装桌面环境
####1.安装gnome
ArchLinux使用的gnome版本是Gnome3，感觉Gnome的体验还是比较好的，画面和动画也很炫，当然还有其他的桌面可以选择，比如KDE，看个人喜好，由于我是用惯了Ubuntu上的Gnome，所以这次安装Arch也干脆使用Gnome，安装图形界面也确实花费了一定时间，整理如下：

```
~ pacman -Syu gnome
~ pacman -Syu gnome-extra
~ paceman -S xorg
~ paceman -S xorg-xinit
~ paceman -S xors-server
~ paceman -S f86-video-vesa
```
以上就完成了Gnome的安装，同时也安装了gdm。gdm是一个script，它提供一个简单的登录界面，来启动图形界面，也就是咱们的Gnome。所以，你需要设置该脚本在系统启动时运行，这样的话，开机就会弹出登录界面。

```
~ systemctl enable gdm #gdm开机启动
~ reboot
```
重启之后就可以直接打开gdm来登录了，这时可能只有root用户可以登录。
登录成功之后可能会发现，无法打开terminal，这可能是因为没有设置语言的原因，请使用以下命令：

```
~ sudo tee /etc/locale.conf <<< LANG=en_US.UTF-8
```
####2.浏览器中文乱码
Arch安装了Chrome之后发现中文乱码，解决方案如下：

```
~ pacman -S wqy-zenhei ttf-fireflysung
```
####3.非root无法使用sudo
添加sudo权限，vim /etc/sudoers,然后添加：
your_name ALL=(ALL)   ALL
### 六、总结
经过以上的不断折腾，总算是装好了Arch，使用之后惊叹于pacman带给我的便捷，package的管理实在是很方便。安装Arch最大的收获不是我最后安装成功，而是在这个过程中遇到的每一个困难，让我能对Linux的一些配置文件、磁盘的分区，以及网络的设置有了一定的了解。
如果有什么写的不对的地方，欢迎指出，谢谢！


