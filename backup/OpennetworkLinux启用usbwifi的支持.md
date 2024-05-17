买了个usbwifi CF-812AC,里面芯片型号应该是88x2bu，这个大把驱动安装教程，但是关键的来了。ONL系统的headers不是类似于标准内核头，可以直接APT安装的。所以费劲千辛万苦去找这个内核头文件。不然modules编译不出来。
千辛万苦找到了deb安装内核头，也能成功编译出*.ko文件，insmod 的时候dmesg出现很多symbol error，一番搜索之后发现，编译内核的时候还需要启用802.11协议，所以在此对整个流程进行记录。
首先一些关键的目录

- 打包系统过程中生成的deb：/home/foxing/opennetworklinux/REPO/buster/packages/binary-amd64 （删除这里的对应deb可以让模块重新被编译，我们这里删除掉原来的onl-kernels*.deb来让我们修改生效）同时这里也是最后生成内核头文件的地方
- 内核编译的配置文件：/home/foxing/opennetworklinux/packages/base/any/kernels/5.4-lts/configs/x86_64-all/x86_64-all.config （这里才是真正生效的内核配置，而不是内核编译目录下的.config
- 内核编译的文件夹：/home/foxing/opennetworklinux/packages/base/amd64/kernels/kernel-5.4-lts-x86-64-all/builds/buster/linux-5.4.40 你可以在这里 make menuconfig 然后把保存的.config去覆盖上面说的all.config来达到图形化配置的目的

下面来看下我们启用的配置：
1.启用802.11相关的协议支持
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/2d6d31c1-d942-452d-bc1e-fd713388972e)
2.启用USB network，wireless lan的支持
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/657f9d0a-e7ae-4674-b636-729830819cea)

之后保存完成后记得按照我上文描述的覆盖x86-all.config，并且删除原来的deb来达到增量编译的效果

将新编出来的系统安装完成后我们使用dksm来进行ko的编译，步骤如下:
```
#先安装我们的内核头文件，目录位于~/opennetworklinux/REPO/buster/packages/binary-amd64onl-kernel-5.4-lts-x86-64-all_1.0.0_amd64.deb，拷贝到设备上dpkg -i 安装
ln -s /usr/share/onl/packages/amd64/onl-kernel-5.4-lts-x86-64-all/mbuilds ./build #链接头文件到安装目录
sudo apt-get update
sudo apt-get -y install wget dkms
git clone https://github.com/cilynx/rtl88x2bu
sudo dkms add ./rtl88x2bu
sudo dkms install -m rtl88x2bu -v 5.8.7.1
sudo modprobe 88x2bu
sudo reboot
```
成功后lsmod,应该可以看到88x2bu的驱动
```
root@localhost:~# lsmod
Module                  Size  Used by
bf_kdrv                24576  0
88x2bu               2592768  0
x86_pkg_temp_thermal    20480  0
gpio_ich               16384  0
root@localhost:~# 
```
之后就是无线网卡的配置了，我这里出现的是：
```
wlx200db0430647: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.225  netmask 255.255.255.0  broadcast 192.168.31.255
        inet6 fda2:95fb:a68b::82c  prefixlen 128  scopeid 0x0<global>
        inet6 fe80::1b63:7c6:d420:201f  prefixlen 64  scopeid 0x20<link>
        inet6 fda2:95fb:a68b:0:1b5e:5811:5e0f:e1d  prefixlen 64  scopeid 0x0<global>
        ether 20:0d:b0:43:06:47  txqueuelen 1000  (Ethernet)
        RX packets 2613553  bytes 3900898666 (3.6 GiB)
        RX errors 0  dropped 53416  overruns 0  frame 0
        TX packets 481522  bytes 52926003 (50.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
这里不推荐任何其余的工具进行网络配置，只推荐nmcli工具
```
#安装
sudo apt update
sudo apt install network-manager
#连接
nmcli device wifi connect SSID_NAME password PASSWORD
#查看状态
root@localhost:~# nmcli device show
GENERAL.DEVICE:                         wlx200db0430647
GENERAL.TYPE:                           wifi
GENERAL.HWADDR:                         20:0D:B0:43:06:47
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     FOXING_5G
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/8
IP4.ADDRESS[1]:                         192.168.31.225/24
IP4.GATEWAY:                            192.168.31.1
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 192.168.31.1, mt = 600
IP4.ROUTE[2]:                           dst = 192.168.31.0/24, nh = 0.0.0.0, mt = 600
IP4.DNS[1]:                             192.168.31.1
IP4.DOMAIN[1]:                          lan
IP6.ADDRESS[1]:                         fda2:95fb:a68b:0:1b5e:5811:5e0f:e1d/64
IP6.ADDRESS[2]:                         fda2:95fb:a68b::82c/128
IP6.ADDRESS[3]:                         fe80::1b63:7c6:d420:201f/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 600
IP6.ROUTE[2]:                           dst = fda2:95fb:a68b::/48, nh = fe80::5233:f0ff:fed7:6610, mt = 600
IP6.ROUTE[3]:                           dst = fda2:95fb:a68b::/64, nh = ::, mt = 600
IP6.ROUTE[4]:                           dst = fda2:95fb:a68b::82c/128, nh = ::, mt = 600
IP6.ROUTE[5]:                           dst = ff00::/8, nh = ::, mt = 256, table=255
IP6.DNS[1]:                             fda2:95fb:a68b::1
```
nmcli会自动dhcp获取到地址非常方便。



