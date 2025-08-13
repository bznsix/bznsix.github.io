# Solidity rawdata的介绍

在Solidity中，函数的调用其实都是通过rawdata来实现的，在与合约交互的过程中，传入一串inputdata，在这个inputdata中就包含了所需要的一切信息，包过需要调用的函数名字，携带进入的参数等。下面我们将进行实验。

```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0;

contract MyToken {
    mapping (address => uint) balances;
    event Transfer(address indexed _from, address indexed _to, uint256 _value);

    function give() external{
        balances[msg.sender] = 10000000;
    }

    function sendCoin(address to, uint amount) public returns(bool sufficient) {
        if (balances[msg.sender] < amount) return false;
        balances[msg.sender] -= amount;
        balances[to] += amount;
        emit Transfer(msg.sender, to, amount);
        return true;
    }

    function getBalance(string calldata function_name,string calldata data) public returns(bytes memory) {
        return abi.encodeWithSignature(function_name,data);
    }

    function return_hash() public returns(bytes memory) {
        return abi.encodeWithSignature("counter_increase(uint256,uint256,uint256)",2,5,7);
    }

    function call_function(address contracts,bytes memory raw) public returns(bool result){
         contracts.call(raw);
         return true;
    }
}

contract counter{
    uint256 public counter = 0;
    uint256 public x = 0;
    uint256 public y = 0;
    uint256 public z = 0;

    function getcounter() public returns(uint256){
        return counter;
    }

    function counter_increase(uint256 x1,uint256 y1,uint256 z1) public{
        counter = counter + 1;
        x = x1;
        y = y1;
        z = z1;
    }
}

//{
	// "0": "bytes: 0xe47b025300000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000"
// }

// {
// 	"0": "bytes: 0xe47b025300000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000"
// }


// 0x553eba1000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005


```

在我们的测试代码中，我们创建了两个合约。一个是简单的计数器，用来检验传参入参是否正确。另一个是用来计算input data，和call 计数器合约function的合约，值得一提的是，每个合约都有个固定的功能叫做call function。这是函数调用的最底层实现。

在solidity中，我们还有个固定函数叫做abi.encodeWithSignature，这个函数的作用就是将函数名字以及函数参数打包成inputdata的形式来给后台调用。

注意：打包时函数名字是字符串形式，且不需要带变量名字，只需要确认参数的类型就好。类似如下：

```js
return abi.encodeWithSignature("counter_increase(uint256,uint256,uint256)",2,5,7);   正确的

return abi.encodeWithSignature("counter_increase(uint256 x,uint256 y,uint256 z)",2,5,7);  错误的
```

同时注意传参类型要与变量类型相匹配，是uint型你直接写数字就好，不需要写字符串。

我们返回个打包成input data的bytes类型看下，如下文所示：

```
0x553eba1000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005
其中前4个字节是通过函数名字生成的函数选择器 0x553eba10
后面的则是依据参数拼的输入参数，输入参数不会经过任何计算，传入的值是多少，那就是多少，例如本文有二个256位的参数，那么就会在函数选择器的后面补上二个256位的参数。传的参数越多，inputdata就会越长。在默认的情况下以太坊EVM会将所有的入参都给补充到32位。
0000000000000000000000000000000000000000000000000000000000000002
0000000000000000000000000000000000000000000000000000000000000005
0000000000000000000000000000000000000000000000000000000000000007
0000000000000000000000005cb11ce550a2e6c24ebfc8df86c5757b596e69c1
0xbf699b4b0000000000000000000000005575406ef6b15eec1986c412b9fbe144522c45ae
0x1e898534000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000000000000000000000000000000000000007
0xa927faa6000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000000000000000000000000000000000000007

```

最后我们尝试使用call方法直接调用目标合约的函数：

```
{
	"address contracts": "0x1c91347f2A44538ce62453BEBd9Aa907C662b4bD",
	"bytes raw": "0x0f958c15000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000000000000000000000000000000000000007"
}
```

可以看到目标函数中的值已经变味了2，5，7

同时solidity中也有打包函数abi.encodePacked，可以将几个参数打包成一串bytes：

例如：

```
abi.encodePacked(bytes12(0), bytes20(address(this)), bytes32(0), bytes32(0))
返回：0x0000000000000000000000008b801270f3e02ea2aaccf134333d5e5a019eff4200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

