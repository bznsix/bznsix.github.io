最近想升级ONL到10，但是ONL这个项目很久没有人维护了遇到了一些问题，这里记录下解决方案
常规的构建流程：
```
#> git clone https://github.com/opencomputeproject/OpenNetworkLinux
#> cd OpenNetworkLinux
#> docker/tools/onlbuilder -10             # enter the docker workspace
#> apt-cacher-ng
#> source setup.env                         # pull in necessary environment variables
#> make amd64                          # make onl for $platform (currently amd64)
```
但是在执行apt-cacher-ng的时候会出现报错，如果在容器中直接apt-install apt-cacher-ng 也会出现dns解析出现错误的问题，而设置这个包缓存器又是编译过程的第一步（没仔细看是否会在后续过程中更改apt源等）所以这就带来了一系列问题，如果不执行这个指令会出现后续的一系列包安装失败，所以这个东西还挺重要，下面直接给出解决方案：

1. 直接下载apt-cacher-ng.deb  wget http://ftp.de.debian.org/debian/pool/main/a/apt-cacher-ng/apt-cacher-ng_3.2.1-1_amd64.deb
2. dpkg -i 加名字，在弹出来的框是否运行http选择yes
3. 继续正常的步骤

注意上述的所有步骤都是在编译ONL的docker中进行的，而不是宿主机

没大问题的话是可以正常走完的

[参考资料](https://groups.google.com/g/opennetworklinux/c/K3Dtt7uWjyk/m/IBkKDZxEAwAJ?pli=1)




