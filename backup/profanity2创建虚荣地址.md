profanity有随机数太简单导致可以被强行爆破私钥的问题，在profanity2里面被修复了，下面简单介绍如何使用profanity2来生成靓号

[项目地址](https://github.com/1inch/profanity2)

使用教程：

1.生成公钥和私钥

`openssl ecparam -genkey -name secp256k1 -text -noout -outform DER | xxd -p -c 1000 | sed 's/41534e31204f49443a20736563703235366b310a30740201010420/Private Key: /' | sed 's/a00706052b8104000aa144034200/\'$'\nPublic Key: /'`

得到结果如下

45432d506172616d65746572733a202832353620626974290a
Private Key: ed95bbeef50839294867ce138034bxxxxx328d600d32db40e625cdeca554d386                                                                                                     

Public Key: 0434bca88471b4b482ce20ce95a35ec533c9fd5285e094897c59c8366a9d67ea75102a2d0def7a3f8a388a25a13db96cd9b2abc6670812929cc45ec08c51cd86bc

2.记住这里的public key，使用profanity2 指令运行你想要的靓号 --matching加靓号形式 -z加publickey **注意去除前面的04**
`./profanity2.arm --matching 0000 -z 34bca88471b4b482ce20ce95a35ec533c9fd5285e094897c59c8366a9d67ea75102a2d0def7a3f8a388a25a13db96cd9b2abc6670812929cc45ec08c51cd86bc`

3.得到结果，输出如下
`Time:     7s Score:  2 Private: 0x000003b2d29425cca1b88cd066b295204197cc862b5f648d057720116798ff11 Address: 0x0000bf5d5a9a33b010183161cc1c284060c41979`
使用privatekey 和 第一步中的生成的privatekey 进行计算，脚本
`hex((ed95bbeef50839294867ce138034bxxxxx328d600d32db40e625cdeca554d386 + 0x000003b2d29425cca1b88cd066b295204197cc862b5f648d057720116798ff11) % 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F)
`
就能得到你想要的靓号了