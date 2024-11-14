Dividendpayingtoken是一种常用在土狗里的分红机制，会根据你持仓的多少、添加流动性的数量来定期进行分红。意在奖励那些持有代币的人。

首先放上一份源码：[源码](https://github.com/Roger-Wu/erc1726-dividend-paying-token/blob/master/contracts/DividendPayingToken.sol)

主要有这些函数：
好的，以下是合约中每个函数及其作用的概述：

### 1. **`function() external payable`**
   - 接收以太币并调用 `distributeDividends()` 函数分发分红。

### 2. **`distributeDividends()`**
   - 将收到的以太币按持币比例分发给所有代币持有者。如果总供应量大于 0 则执行，并触发 `DividendsDistributed` 事件。

### 3. **`withdrawDividend()`**
   - 允许调用者提取其累积的分红金额，并触发 `DividendWithdrawn` 事件。

### 4. **`dividendOf(address _owner)`**
   - 查看指定地址可以提取的分红金额（以 wei 为单位）。

### 5. **`withdrawableDividendOf(address _owner)`**
   - 查看指定地址当前可提取的分红总金额。

### 6. **`withdrawnDividendOf(address _owner)`**
   - 查看指定地址已提取的分红金额。

### 7. **`accumulativeDividendOf(address _owner)`**
   - 查看指定地址累计获得的分红总金额，包括已提取和未提取的部分。

### 8. **`_transfer(address from, address to, uint256 value)`**
   - 内部函数，用于在地址间转移代币，同时更新 `magnifiedDividendCorrections` 以确保分红保持不变。

### 9. **`_mint(address account, uint256 value)`**
   - 内部函数，用于向指定地址铸造新代币，同时更新 `magnifiedDividendCorrections` 以确保分红保持不变。

### 10. **`_burn(address account, uint256 value)`**
   - 内部函数，用于销毁指定地址的代币，同时更新 `magnifiedDividendCorrections` 以确保分红保持不变。

我们在这里重点看下withdrawableDividendOf，这代表的是用户可以提取分红的数量他由accumulativeDividendOf（用户可提取分红总金额 ）-  withdrawnDividendOf（用户已经提取分红金额）组成。

而accumulativeDividendOf 又由
```
return magnifiedDividendPerShare.mul(balanceOf(_owner)).toInt256Safe()
      .add(magnifiedDividendCorrections[_owner]).toUint256Safe() / magnitude; 
```
计算而来

前面很好理解 大概就是每一份balance * 每一份奖励的金额。但是为什么有一个magnifiedDividendCorrections呢？

理解这个问题的关键在于 `magnifiedDividendCorrections` 的用途，它实际上在公式中用于 **抵消分红**。我们来详细解释其中的逻辑。

### 分红计算公式回顾

在合约中，用户的 **累计分红** 通过以下公式计算：

```
return magnifiedDividendPerShare.mul(balanceOf(_owner)).toInt256Safe()
      .add(magnifiedDividendCorrections[_owner]).toUint256Safe() / magnitude; 
```

这意味着：

- **`magnifiedDividendPerShare * balanceOf(user)`** 是用户基于持有代币数量应得的总分红份额。
- **`magnifiedDividendCorrections[user]`** 是用于抵消或调整用户分红的修正值。

### 负向修正的原理

在分红计算中，**当 `magnifiedDividendCorrections` 增加时，实际上会减少用户的总分红**。这是因为 `magnifiedDividendCorrections` 其实是以 **负数**的方式存储的。

#### 1. 为什么 `magnifiedDividendCorrections` 是负数
`magnifiedDividendCorrections` 的初始值为 0，当用户收到分红代币时，系统会 **减少** 该值，表示用户有资格获取更多分红；当用户减少代币余额（如转出或销毁），系统会 **增加** 该值，从而抵消一部分分红。

换句话说，**`magnifiedDividendCorrections` 实际上是用来扣除（抵消）一部分分红的**，所以当它增加时，代表对用户的分红做了更大的扣除，导致分红减少。

#### 2. 具体流程示例

假设 `magnifiedDividendPerShare` 是一个固定的分红份额，用户 `A` 最初持有 `100` 个代币，并没有进行任何转账操作。

- 当有新的分红增加时，`magnifiedDividendPerShare` 增加，用户的 `balanceOf(user)` 确定，所以 `magnifiedDividendCorrections` 不变。
- 如果用户 `A` 将部分代币（如 50 个）转出，那么 `magnifiedDividendCorrections[user]` 会增加，表示未来计算中应减少一部分分红份额，因为 `A` 不再持有这 50 个代币的分红权。

### 总结

- **`magnifiedDividendCorrections` 增加时，实际分红会减少**。这是因为 `magnifiedDividendCorrections` 中的值本质上是一个抵消项，增加这个值意味着对分红的扣减增加。


在初始化mint token的时候    magnifiedDividendCorrections[account] = magnifiedDividendCorrections[account]
      .sub( (magnifiedDividendPerShare.mul(value)).toInt256Safe() ); 这个值会被设置为0，刚好等于你有的token 加上一个校正的负值。

这样用户就不会享受到分红合约中历史时刻累计的ETH奖励。

只有当分红合约收到了资金的时候，触发了  
```
function distributeDividends() public payable {
    require(totalSupply() > 0);

    if (msg.value > 0) {
      magnifiedDividendPerShare = magnifiedDividendPerShare.add(
        (msg.value).mul(magnitude) / totalSupply()
      );
      emit DividendsDistributed(msg.sender, msg.value);
    }
  }
```
而这个值会增加pershare的值，从而导致奖励计算中的magnifiedDividendPerShare.mul(balanceOf(_owner)).toInt256Safe() 变大，而magnifiedDividendCorrections[_owner]).toUint256Safe() / magnitude; 是记录的用户mint的那一刻的magnifiedDividendPerShare，从而使两者之间产生了差值，开始分红。

总而言之，挺巧妙的设计，同时需要一些数学头脑。
