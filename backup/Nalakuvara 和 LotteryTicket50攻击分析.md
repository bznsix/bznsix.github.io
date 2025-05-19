这个曾经审计过，但是没看出来问题。当时看到的是一个mint lottery的合约好像，没有找到真正的漏洞点。在这里分析下出现问题的原因。

首先[攻击Tx](https://app.blocksec.com/explorer/tx/base/0x16a99aef4fab36c84ba4616668a03a5b37caa12e2fc48923dba4e711d2094699)

第一步 攻击者先用280w刀的USDC来兑换Nala token

<img width="1653" alt="Image" src="https://github.com/user-attachments/assets/f09138eb-360d-444e-995f-dc1c1251014e" />

然后调用了LotteryTicket的transferToken来mint lottery ticket，这里会mint 很多ticket然后拿转走你的usdc来添加流动性，相当于是LP token。

```
function transferToken(uint  amount) public returns (bool) {
        IUniswapV2Router02  swapRouter = IUniswapV2Router02(ROUTER_ADDRESS);
        
        require(amount > 0, "Amount must more than 0 USDT");
        require(amount % MIN_DEPOSIT == 0, "Amount must be a multiple of 50 USDT");
        uint allowmount= coinUsdt.allowance(msg.sender,address(this));
        require(allowmount>=amount, "Insufficient authorization limit");
        
        // 转账 USDT 到合约
        require(coinUsdt.transferFrom(msg.sender,address(this),amount), "USDT transfer failed");

        uint ticket_count=amount/MIN_DEPOSIT;
        
        require(coinTicket.mint(msg.sender,ticket_count * 10 ** 6), "Ticket mint failed");

        uint amountIn=amount*50/100;
        address[] memory path = new address[](2);
            path[0] = tokenUSDT;
            path[1] = tokenNATA;
        coinUsdt.approve(ROUTER_ADDRESS, amount);
        swapRouter.swapExactTokensForTokensSupportingFeeOnTransferTokens(
            amountIn,
            1,
            path,
            address(this),
            block.timestamp+600
        );
   
        uint amountADesired=amountIn*997/1000;
        // 批准 Uniswap 路由合约转移 U 和 NATA 代币用于添加流动性
        coinUsdt.approve(ROUTER_ADDRESS, amount);
        coinNata.approve(ROUTER_ADDRESS, coinNata.totalSupply());

        // 3. 调用Pair合约获取储备数据
        address token0 = IUniswapV2Pair(pairAddress).token0();


        if(token0==tokenUSDT){
            ( reserveUSDT,  reserveNATA,) = IUniswapV2Pair(pairAddress).getReserves();
        }else {
             (reserveNATA, reserveUSDT,) = IUniswapV2Pair(pairAddress).getReserves();
        }
       
        //uint 
        amountBDesired = (amountADesired * reserveNATA) / reserveUSDT;
        if(flag1){
            swapRouter.addLiquidity(
                tokenUSDT, 
                tokenNATA, 
                amountADesired, 
                amountBDesired, 
                1, 
                1, 
                address(this),
                block.timestamp+600
                );
        }

        return true;
    }
```

之后调用了DestructionOfLotteryTickets 来销毁第一步mint 给你的ticket（相当于LP）

**问题原因**

让我们看两段关键的代码
```
1.
        // 转账 USDT 到合约
        require(coinUsdt.transferFrom(msg.sender,address(this),amount), "USDT transfer failed");

        uint ticket_count=amount/MIN_DEPOSIT;
        
        require(coinTicket.mint(msg.sender,ticket_count * 10 ** 6), "Ticket mint failed");

2.
        uint ticket_count=_amountTickets/MIN_TICKET;
        uint amountUSDTALL=ticket_count*MIN_DEPOSIT;
        uint amountUSDT=amountUSDTALL/2*997/1000;
```

第一步中的mint ticket 看的是你提供了多少USDC，第二部中的 destroy ticket 看的是你持有多少ticket，他并不计算这些ticket对应了多少的value 的USDC。

所以核心的攻击思路就是--> 先拉高整个Nala-USDC的pair交易对的价格，逼着lottery合约以高价格添加流动性（这时候合约里几乎没有了Nala）你可以抽象成只添加了USDC进pair。最后换回Nala，destroy ticket。

整个流程可以抽象成为一个三明治攻击，把lottery合约的添加流动性夹在了中间。从而获取了利润。

那么问题就来了：常规的三明治攻击都是夹杂着合约自己就有的USDC，或者是第三方的资金。但是这里用于添加流动性的是我们自己的USDC，利润从哪里来的？

我的解释是，用于添加流动性的usdc拉高了整个池子的平均份额，从而在分批次减少流动性的时候能够获取到更多的利润。
