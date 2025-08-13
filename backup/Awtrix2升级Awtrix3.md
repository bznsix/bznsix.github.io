原来就一直有个像素灯，之前是通过上位机和下位机通信控制的，但是上次打开控制网页的时候出现了当前版本已经停止支持的选项，遂上github查看发现从ESP8266重构到了ESP32，不需要上位机控制了。
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/48682fb6-e7b3-4411-aae9-26e736d4fa2f)
国外友人也很友善的提供了二代到3代升级的版本，只需要购买个ESP32mini即可，某宝下单，花费15大洋。
今日到货，两个对比，确实差不多大
![6149edd78c9232a4f0f2b118b488937](https://github.com/bznsix/bznsix.github.io/assets/38829902/37df3d28-8a85-4c92-8b94-44d155a9b86c)
先刷入固件，作者提供了网页刷入支持，可以直接从web选择你的串口进行刷入，我们选择从Awtrix2升级刷入，[地址:](https://blueforcer.github.io/awtrix3/#/flasher)
![9416b3a7158d3f8d777e51e461b4c3b](https://github.com/bznsix/bznsix.github.io/assets/38829902/66926f76-afed-4846-830f-886080c10bc3)
进行一些朴实的焊接后再翻开来对比：
![83d4da9718104ba749f4ae4d717ab80](https://github.com/bznsix/bznsix.github.io/assets/38829902/f5d273fe-6a1e-4283-86be-6d71f9f78cee)
可以看到内部的脚位差不多一致，之后拔掉原来的8266，插入新的ESP32开机：
![e3f366eaa8a819878055ce829d0d26f](https://github.com/bznsix/bznsix.github.io/assets/38829902/08f92c3a-12f4-4cad-a9fd-6b5dec518aca)
你会发现屏幕是一坨乱码，没关系我们先链接上设备创建的默认WIFI：awtrix_xxxx,密码：12345678 进入file_manger模块，在根目录创建一个dev.json，内容如图所示，上传完成后重启
![1fa8680125851fd2e90ce98f2bdce36](https://github.com/bznsix/bznsix.github.io/assets/38829902/cc07abf7-433d-4cde-936e-c25400189cfd)
出现图案了
![32d085dc0696bb8e77de905cb1f0f5b](https://github.com/bznsix/bznsix.github.io/assets/38829902/ff59a470-027a-4137-97f2-5c68c898d30a)
最后买个APP对比下：
![6406db8e6b2918692555b90843918d3](https://github.com/bznsix/bznsix.github.io/assets/38829902/2f65421c-780a-4fa0-a624-872e530721c7)
新老对比：
![7c1a3bd5ac640240ade3561679372d3](https://github.com/bznsix/bznsix.github.io/assets/38829902/5e44f38b-fa36-45b4-85a2-f53c8b040409)
![0e39bc63b3bea92dd4a51c4a10ea392](https://github.com/bznsix/bznsix.github.io/assets/38829902/87744a8a-6f91-422b-8710-68e158098721)
说实话感觉新的动画不是很多，而且要收费==，还是老的好，不用上位机对我来说也不是什么大的吸引点，因为我有个24h开机的小主机，多跑一个docker的事情。果真创造工作量的做好方法就是重构哈哈。就这样。


