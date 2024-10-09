经常攻击合约的小伙伴们都知道，经常有一些合约会有任意call的漏洞，允许你自定义调用calldata，常用的攻击链条就是构造包含transferfrom的calldata，偷偷把之前授权过危险合约的地址的钱给转过来。
像是这个漏洞[tx](https://x.com/TenArmorAlert/status/1843632599169900558)
那么，如果我们想要复制这种交易，或者是批量查询谁事先授权过危险合约怎么办呢？一个个翻交互TX吗？No！这里给出一个解决方案。
众所周知，你在Approve一个地址spend你的代币的时候是会在区块链上发出event事件的，而Dune正好可以查询这些事件，一个标准的Approve事件如下图所示：
![image](https://github.com/user-attachments/assets/f5758ef9-f1d8-471f-9a29-a37dd9a94455)

1. Topics0:这里是事件的签名hash,只要是approve事件，那就是这个hash.