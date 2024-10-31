经常攻击合约的小伙伴们都知道，经常有一些合约会有任意call的漏洞，允许你自定义调用calldata，常用的攻击链条就是构造包含transferfrom的calldata，偷偷把之前授权过危险合约的地址的钱给转过来。
像是这个漏洞[tx](https://x.com/TenArmorAlert/status/1843632599169900558)
那么，如果我们想要复制这种交易，或者是批量查询谁事先授权过危险合约怎么办呢？一个个翻交互TX吗？No！这里给出一个解决方案。
众所周知，你在Approve一个地址spend你的代币的时候是会在区块链上发出event事件的，而Dune正好可以查询这些事件，一个标准的Approve事件如下图所示：
![image](https://github.com/user-attachments/assets/f5758ef9-f1d8-471f-9a29-a37dd9a94455)

1.Topics0:这里是事件的签名hash,只要是approve事件，那就是这个hash.
2.Owner
3.Spender

那么很简单，我们只需筛选出来topic0是approve事件，spender是被攻击合约的，就能简单得到了
![image](https://github.com/user-attachments/assets/5366d258-ea53-4205-bd72-978fda80ce0f)

0xb69a就是我们上面攻击的受害者

给出查询代码
```SQL
SELECT *
FROM bnb.logs
WHERE topic0 = 0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925
AND topic2 = 0x000000000000000000000000068ba5d0540e27b39c71a00a1c0c1e669d62dc10
LIMIT 20;
```

补充一个冷知识：合约的字节码中会有函数的signature,所以我们可以查找函数的4字节slector code:
```
SELECT
  *
FROM ethereum.creation_traces
WHERE
  TRY_CAST(code AS VARCHAR) LIKE '%689368%'
LIMIT 20
```

结果示意图：
![image](https://github.com/user-attachments/assets/5a7a9213-dbbb-44bd-88c9-3bd5515e6569)

这样可以查找到具有特定函数名称的合约
