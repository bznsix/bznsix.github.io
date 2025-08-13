昨天一个攻击引起了我的注意，交易[TX](https://app.blocksec.com/explorer/tx/bsc/0x49ca5e188c538b4f2efb45552f13309cc0dd1f3592eee54decfc9da54620c2ec)
,本质原因是未经认证的输入，可以导致用户的输入calldata被合约执行，这很常见了。
黑客传入的calldata 经过decode后调用了1inch router的unoswap函数，我们看下函数定义：
```
    /// @notice Performs swap using Uniswap exchange. Wraps and unwraps ETH if required.
    /// Sending non-zero `msg.value` for anything but ETH swaps is prohibited
    /// @param recipient Address that will receive swapped funds
    /// @param srcToken Source token
    /// @param amount Amount of source tokens to swap
    /// @param minReturn Minimal allowed returnAmount to make transaction commit
    /// @param pools Pools chain used for swaps. Pools src and dst tokens should match to make swap happen
    function unoswapTo(
        address payable recipient,
        IERC20 srcToken,
        uint256 amount,
        uint256 minReturn,
        uint256[] calldata pools
    ) external payable returns(uint256 returnAmount) {
        return _unoswap(recipient, srcToken, amount, minReturn, pools);
    }
```
前面几个参数都很好理解，但是这个最后一行的calldata pools就很让人费解了，常规来说，这一段应该是个池子的代码，但是这里却采用了uint256类型的数组传入变量，考虑到1inch是个聚合器协议，所以这一块很可能是为了他实现路由的操作协议。同时为了节省gas,在函数里面大量使用了汇编。导致可读性很差。
在查看了POC后，发现也只是单纯抄袭了黑客的传参，并没有解释含义。
但是在观察调用后，发现了在unoswap执行后会去调用pancakeswap的swap函数去进行代币->WBNB的交换，所以这个传参大概率是含有Pair的地址的。
对10进制的uint256进行转换得到：
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/a197cab4-f9d8-49ab-bdff-0f8b9cd9185d)
发现数组主要有以下几部分构成：
0x4000000000000000，3b74a460，61eb789d75a95caa3ff50ed7e47b96c132fec082
其中最后一部分刚好是BTCB-WBNB的池子地址
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/4b98911f-37ae-4ae9-b07c-0b5ba59ddc62)
至于前面两部分具体代表了什么等我再深入看下solidity的汇编再说。
