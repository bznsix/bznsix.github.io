挺简单的一个POC，但是其中蕴含的数学分析还是挺有意思的，分析一下提升下自己对数字的敏感性。
链接：[TX](https://app.blocksec.com/explorer/tx/bsc/0x5e694707337cca979d18f9e45f40e81d6ca341ed342f1377f563e779a746460d)
合约地址:(https://bscscan.deth.net/address/0xbe779d420b7d573c08eee226b9958737b6218888)
代币本身是一个314类型代币，即由合约本身进行一个价格的计算

有问题的代码片段：
```
  function buy() internal {
    require(tradingEnable, 'Trading not enable');

    uint256 swapValue = msg.value;

    //这里有点细节，进到这个函数的时候address(this).balance已经加上msg.value了。
    uint256 token_amount = (swapValue * _balances[address(this)]) / (address(this).balance);

    require(token_amount > 0, 'Buy amount too low');

    uint256 user_amount = token_amount * 50 / 100;
    uint256 fee_amount = token_amount - user_amount;

    _transfer(address(this), msg.sender, user_amount);
    _transfer(address(this), feeReceiver, fee_amount);

    emit Swap(msg.sender, swapValue, 0, 0, user_amount);
  }

  function sell(uint256 sell_amount) internal {
    require(tradingEnable, 'Trading not enable');

    uint256 ethAmount = (sell_amount * address(this).balance) / (_balances[address(this)] + sell_amount);

    require(ethAmount > 0, 'Sell amount too low');
    require(address(this).balance >= ethAmount, 'Insufficient ETH in reserves');

    uint256 swap_amount = sell_amount * 50 / 100;
    uint256 burn_amount = sell_amount - swap_amount;

    _transfer(msg.sender, address(this), swap_amount);
    _transfer(msg.sender, address(0), burn_amount);

    payable(msg.sender).transfer(ethAmount);

    emit Swap(msg.sender, 0, sell_amount, ethAmount, 0);
  }
  ```
  简单可以看到，每次买入或者卖出代币的时候，将会扣减50%的税进行燃烧。
  买入的数量 或者是卖出的价格 均由代币合约的持币数量和BNB数量决定。
  这是和uniswapv2差不多的一个价格计算模式，恒定乘积公式，双曲线模式。买的越多，价格越高。
  但是问题出现在后半部分的burn_amount操作上。
  你可能会觉得，这不就是个带税的ERC20吗？很多shit coin都有这种操作，而且uniswap也支持收税的代币swap，哪里会有问题呢？
  但是注意时间点，uniswap计算收税代币的兑换比例是以最终pair收到了多少代币来计算的，而这里则是在**收税前计算你能换出来BNB数量**，给你打钱了再扣税，这时候已经为时已晚了。这就是利润的来源了。