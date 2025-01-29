### 前言
为了更快的筛选土狗，需要搭建一个本地节点，fast node的最前面的128个块是全数据的也够我们使用了，而且占用的空间很小，大概只需要500G左右可以说成本是相当的低了。

### 准备工作
下载geth
```
# Linux
wget   $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep geth_linux |cut -d\" -f4)
mv geth_linux geth
chmod -v u+x geth
```
下载配置文件
```
wget   $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep mainnet |cut -d\" -f4)
unzip mainnet.zip
```
下载snap快照,跟着48的指示应该一看就会，注意下载fast那个最小的那个，这样的话也不用裁剪数据了
```
https://github.com/48Club/bsc-snapshots
```
### 运行
全部解压完成后运行指令
```
./geth --tries-verify-mode none --config ./config.toml --datadir ./bsc_node/geth  --cache 8000 --rpc.allow-unprotected-txs --history.transactions 0
```
注意这里选用的--datadir应该是包含了chaindata的上层文件夹，如果你文件夹目录没选对那你的快照基本上没用，还是从头开始同步。

给出我的文件夹目录示例：
``` └── geth
      ├── bsc.log -> /home/foxing/node/bsc_node/geth/bsc.log.2025-01-29_15
      ├── chaindata
      ├── geth
      ├── geth.ipc
      └── keystore
```
可以看到我这里的geth文件夹下含有chaindata文件夹，所以这里给定的datadir是父文件夹geth。

### 补充
[官方教程](https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/fast_node/#download-the-pre-build-binaries-from-release-page-or-follow-the-instructions-below)
裁剪数据指令
```
./geth snapshot insecure-prune-all --datadir ./node  ./genesis.json
```

