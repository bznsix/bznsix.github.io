蛮经典的一个CTF，地址[https://onlypwner.xyz/challenges/5](url)
这里选择采用Foundry来完成解题，一般来讲，每一关你点击Launch都会生成一个给你的节点，你要攻击的合约地址，以及一个你答题用的私钥地址，如图所示：
![image](https://github.com/user-attachments/assets/6853dc5a-ebf2-4fba-9d14-d09b03ac5c3b)
这些记下来，我们后面会用到
接着我们使用github上一个答题模板来配置，名字叫onlypwner-foundry，可以自行搜索下
下载完成后，记得使用git submodule update --progress --init来更新目录lib下的一些foundry脚本合约
然后新建.env文件，里面用上了我们上面截图的东西
![image](https://github.com/user-attachments/assets/ecce12ab-2b2a-4328-b43b-f844ba38e651)
之后再在challenges文件夹下，你对应的关卡里面的script文件夹下创建Solve.sol解题脚本，以第一关为例（这个很简单，就是存粹提款）
![image](https://github.com/user-attachments/assets/16ba8954-e363-43a9-8525-2e8f65e15361)
这里面会用vm.envxxx去读取我们配置的环境变量，之后我们使用startBroadcast将操纵广播上链。
值得一提的是，我们执行的脚本需要加上--with-gas-price=0 --priority-gas-price 0，因为这个节点是0gas的，你需要指定使用0gas并且高优先级的gas也为0，不然高优先级的gas比低优先级的要低会出错。完整的指令：
```
forge script /home/foxing/onlypwner-foundry/challenges/Freebie/script/Solve.sol --broadcast --with-gas-price=0 --priority-gas-price 0
```


