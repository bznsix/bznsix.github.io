为什么要提到这点呢，因为区块链上如果进了公共的pool的话你的交易就很容易被夹，特别是你这个交易还能产生利润的话，所以我们希望交易具有确定性。这就诞生了隐私节点的概念。
隐私节点，顾名思义，他会保护你的隐私，不会将你发送给他的交易广播出去，但是代价就是上链速度慢以及隐私节点自身存在作恶风险。以BSC上的隐私节点举例，将网络RPC地址设置为[](https://rpc-bsc.48.club)即可启用隐私节点。
同时，为了防止在某些过程出现问题导致这个交易被泄露到了公共内存池中，我们可以使用block.coinbase以及gas price等参数来限定打包我们交易的人。
例如：BNB48的验证者的矿工地址为：0x72b61c6014342d914470eC7aC2975bE345796c2b
           BNB48 打包交易的gas price一般为1gwei
用这两个条件我们就可以限定我们的交易来被特定的验证者打包

实验：
创建合约：
```
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;


contract My{
    uint256 public a = 0;

    function check_gas_price() public  returns (bool) {
        require(tx.gasprice == 1 gwei,'Gas price not equal');
        a ++;
        return true;
    }

    function check_validitor() public  returns (bool) {
        require(block.coinbase == address(0x72b61c6014342d914470eC7aC2975bE345796c2b),'Gas price not equal');
        a ++;
        return true;
    }
}
```
这个合约发布成功后，check_gas_price的功能会检查只有当你的gasprice为1时才增加a,而check_validitor则是只有验证者为bnb48_club才增加a
让我们来验证结果：
当验证者为：Validator: Avengers 时，合约功能执行失败。为什么使用BNB48的RPC却会出现不同的验证者呢？这个原因是因为BNB48并不会自己打包所有的交易，他也会把交易给他信任的合作伙伴。在gitbook里面也有所提到：[gitbook](https://docs.48.club/buidl/infrastructure/bsc-rpc/front-run-protection)
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/f12e7651-7834-478b-ad00-c6513a5062d8)
当验证者为：Validator: 48 Club 时，合约功能执行成功。
![image](https://github.com/bznsix/bznsix.github.io/assets/38829902/5c102c38-38fe-4258-b172-50b09e3de0ab)
总的来说，BNB48是个很不错的RPC，低廉的价格，可靠的保证，推荐大家都用用。
