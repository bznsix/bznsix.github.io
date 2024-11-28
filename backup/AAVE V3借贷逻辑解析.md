引言：先摆上一张图在这：
<img width="767" alt="image" src="https://github.com/user-attachments/assets/7285a792-646e-4711-bdee-3c0a7c38daa6">
再放个参考链接：[解析](https://mirror.xyz/0x4339eA909BF03E7328AC14c75aF0940c082EA994/p1UxvRfJ_cTy-fZcEi66-AkEsb8jifZ9hNz5RUVptvU)

AAVE V3中最重要的就是它的pool合约，我们也只需要关注这个合约就行。
```
contract Pool is PoolStorage {
    // 存款相关
    function supply(
        address asset,
        uint256 amount,
        address onBehalfOf,
        uint16 referralCode
    ) external;

    // 借款相关
    function borrow(
        address asset,
        uint256 amount,
        uint256 interestRateMode,
        uint16 referralCode,
        address onBehalfOf
    ) external;

    // 清算相关
    function liquidationCall(
        address collateralAsset,
        address debtAsset,
        address user,
        uint256 debtToCover,
        bool receiveAToken
    ) external;

    // 闪电贷
    function flashLoan(
        address receiverAddress,
        address[] calldata assets,
        uint256[] calldata amounts,
        uint256[] calldata interestRateModes,
        address onBehalfOf,
        bytes calldata params,
        uint16 referralCode
    ) external;
}
```
在aave v3中所有的资产都由这一个pool合约管理，但是pool合约中的资产可以通过一些控制位来做隔离，防止单一坏账影响到整个池子。资产的配置如下：
```
contract Pool {
    // 通过 mapping 来管理不同资产的配置
    mapping(address => DataTypes.ReserveData) internal _reserves;
    
    // 每个资产都有自己的配置
    struct ReserveData {
        //对应的 aToken 合约
        address aTokenAddress;
        //稳定利率债务 token
        address stableDebtTokenAddress;
        //浮动利率债务 token
        address variableDebtTokenAddress;
        //利率策略合约地址
        address interestRateStrategyAddress;
        // ... 其他配置
    }
}
```
每个资产有自己的:

1. 利率策略
2. 清算阈值
3. 抵押因子
4. 借贷限额

在aave中，你deposit资产会给你对应的atoken，你borrow了资产会根据固定利率模型和浮动利率模型个你mint对应的debt token。
```
Network: Ethereum Mainnet
└── Pool Contract
    ├── USDC Reserve
    │   ├── aUSDC
    │   ├── Variable Debt USDC
    │   └── Stable Debt USDC
    ├── ETH Reserve
    │   ├── aETH
    │   ├── Variable Debt ETH
    │   └── Stable Debt ETH
    └── WBTC Reserve
        ├── aWBTC
        ├── Variable Debt WBTC
        └── Stable Debt WBTC
 ```
 资产的配置文件如下：
 ```
 contract Pool {
    // address: 借贷资产的合约地址
    // 比如 USDC 的合约地址: 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
    mapping(address => DataTypes.ReserveData) internal _reserves;

    struct ReserveData {
        //每个资产的配置数据
        uint256 configuration;
        //流动性指标
        uint128 liquidityIndex;
        uint128 currentLiquidityRate;
        //可变利率指标
        uint128 variableBorrowIndex;
        uint128 currentVariableBorrowRate;
        //固定利率指标
        uint128 currentStableBorrowRate;
        //时间戳
        uint40 lastUpdateTimestamp;
        //对应的 aToken 地址 (如 aUSDC)
        address aTokenAddress;
        //固定利率债务 token 地址
        address stableDebtTokenAddress;
        //可变利率债务 token 地址
        address variableDebtTokenAddress;
        //利率策略合约地址
        address interestRateStrategyAddress;
        //储备金 id
        uint8 id;
    }
}
```
如果你deposit了对应的资产，资产的流向是：
```
用户钱包
└── 1000 USDC
    │
    ▼
aUSDC 合约
└── 存储实际的 1000 USDC
    │
    ▼
用户钱包
└── 获得 1000 aUSDC（代表你的存款份额）
```
所以我们只需要去对应的atoken合约下看他持有的资产就能看到对应的数据。


想要获取池子的信息我们可以主要看两个东西
1.AaveProtocolDataProvider  这里提供了所有的atokens信息，池子借贷多少资产多少等信息。
2.PoolAddressesProvider  这里提供了具体管理的池子信息，使用的预言机地址等。

预言机的实现一般是这样的：
```
  /// @inheritdoc IPriceOracleGetter
  function getAssetPrice(address asset) public view override returns (uint256) {
    AggregatorInterface source = assetsSources[asset];

    if (asset == BASE_CURRENCY) {
      return BASE_CURRENCY_UNIT;
    } else if (address(source) == address(0)) {
      return _fallbackOracle.getAssetPrice(asset);
    } else {
      int256 price = source.latestAnswer();
      if (price > 0) {
        return uint256(price);
      } else {
        return _fallbackOracle.getAssetPrice(asset);
      }
    }
  }
```
会根据asset资产的地址获取对应的assetsSources[asset]地址，从而调用对应的latestAnswer函数获取最新的价格。