同理，为了理解Compound的整体架构，我们从他的borrow/repay入手，出于0市场攻击等达成的条件太苛刻。我们在此不再介绍。

存款流程
```
用户 -> 存入DAI -> CErc20合约 
     <- 获得cDAI <- 
```

借款流程
```
用户 -> 存入ETH -> CEther合约
     <- 获得cETH <-
     -> 借出DAI -> Comptroller检查 -> CErc20合约放款
```
借款的直接入口：
```
// CToken.sol
function borrow(uint borrowAmount) external returns (uint) {
    // 1. 累计利息
    accrueInterest();
    
    // 2. 调用内部借款函数
    return borrowInternal(borrowAmount);
}
```
核心的借款逻辑
```
function borrowInternal(uint borrowAmount) internal nonReentrant returns (uint) {
    // 检查市场是否激活
    require(accrualBlockTimestamp == getBlockTimestamp(), "Accrue interest failed");
    
    // 调用 Comptroller 检查借款条件
    uint allowed = comptroller.borrowAllowed(address(this), msg.sender, borrowAmount);
    require(allowed == 0, "Borrow not allowed");
    
    // 检查市场是否有足够的现金
    require(getCashPrior() >= borrowAmount, "Insufficient cash");
    
    // 更新借款人状态
    accountBorrows[msg.sender].principal = borrowAmount;
    accountBorrows[msg.sender].interestIndex = borrowIndex;
    totalBorrows = totalBorrows + borrowAmount;
    
    // 转移资产给借款人
    doTransferOut(msg.sender, borrowAmount);
    
    emit Borrow(msg.sender, borrowAmount, accountBorrows[msg.sender].principal, accountBorrows[msg.sender].interestIndex);
    return 0;
}
```
Comptroller的借款允许检查
```
function borrowAllowed(address cToken, address borrower, uint borrowAmount) external returns (uint) {
    // 验证市场
    require(markets[cToken].isListed, "Market not listed");
    require(!borrowGuardianPaused[cToken], "Borrow is paused");
    
    // 检查用户是否在市场中
    require(markets[cToken].accountMembership[borrower], "Not in market");
    
    // 检查用户的流动性
    (Error err, , uint shortfall) = getHypotheticalAccountLiquidityInternal(
        borrower,
        CToken(cToken),
        0,
        borrowAmount
    );
    require(err == Error.NO_ERROR, "Error calculating liquidity");
    require(shortfall == 0, "Insufficient liquidity");
    
    // 其他检查...
    
    return uint(Error.NO_ERROR);
}
```
整体的借款流程
```
用户
  │
  ▼
CToken.borrow()
  │
  ├─► accrueInterest()  // 更新利息
  │
  ├─► borrowInternal()
  │     │
  │     ├─► Comptroller.borrowAllowed()  // 检查借款条件
  │     │     │
  │     │     ├─► 检查市场状态
  │     │     ├─► 检查用户资格
  │     │     └─► 检查用户抵押品充足性
  │     │
  │     ├─► 更新借款状态
  │     └─► 转移资产
  │
  └─► 发出事件
```
要进行借款的话：
```
要进行借款，用户需要：
先存入抵押品（调用相应 CToken 的 mint 函数）
确保已进入相关市场（通过 Comptroller.enterMarkets）
然后才能调用目标 CToken 的 borrow 函数进行借款
```