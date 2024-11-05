首先说几个很重大的误区：
一定不能--syncmode 设置为full,这样会从第一个区块开始慢慢同步，**一定要设置为snap**，这样才会从你导入的缓存开始同步
你看到了这样的日志就说明在从缓存同步了
![image](https://github.com/user-attachments/assets/762e7a8f-3fca-451f-82a2-b3684d6217c2)
最后附上我自己的同步指令：
```
/home/foxing/bnb_fullnode/geth --config /home/foxing/bnb_fullnode/config.toml --datadir /home/foxing/bnb_fullnode/data/ --cache 24000 --http --http.api 'web3,eth,net,debug,personal' --rpc.allow-unprotected-txs --txlookuplimit 0 --syncmode=snap --snapshot=true --tries-verify-mode=none --diffblock=5000 --maxpeers 500 --maxpendpeers 200 --ws --ws.port 8545 --ws.addr localhost --ws.origins=*
```
几篇很重要的参考：
[1](https://github.com/bnb-chain/bsc/issues/1193)
[2](https://learnblockchain.cn/article/5324)

