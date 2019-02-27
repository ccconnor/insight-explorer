1. 安装nvm
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
安装完成后重新连接终端，command -v nvm有回应则成功。
参考：https://github.com/creationix/nvm#install-script
```
2. 安装Node.js v4 LTS
```
nvm install v4
```
3. 安装依赖
```
apt-get install libzmq3-dev build-essential
```
4. 安装Bitcore
```
npm install -g bitcore
注意：不能以sudo或root权限安装
```
5. 运行Bitcore
```
bitcored
此时你不会运行成功，会提示bitcore-lib有多个实例，这是因为bitcore的每个子模块中都带着bitcore-lib，只需要一个即可。find $HOME/.nvm/versions/node/v4.9.1/lib/node_modules -name "bitcore-lib"（注意路径换成自己的路径）找到所有的bitcore-lib，只保留一个路径最短的bitcore-lib，其他都删掉。
再次运行bitcored即可成功，默认会在$HOME/.bitcore目录下启动一个节点，并带有insight api和insight ui，通过ip:3001/insight可访问。
```

以上步骤启动的是全局节点，可以作为比特币的浏览器，如果要启动其他服务或者搭建分叉币浏览器，可以在默认目录下改配置，也可以新建一个自定义节点。
下面以新建节点为例。

1. 新建节点
```
bitcore create nodename
```
2. 安装服务
```
cd nodename
bitcore install insight-api insight-ui
```
3. 尝试运行
```
bitcored运行
仍然会碰到bitcore-lib重复的坑，继续删之，运行成功后停掉。接下来要配置分叉币相关。
```
4. 配置网络
```
vi bitcore-node.json
{
  "network": "livenet",
  "port": 3001,
  "services": [
    "bitcoind",
    "insight-api",
    "insight-ui",
    "web"
  ],
  "servicesConfig": {
    "bitcoind": {
      "spawn": {
        "datadir": "./data",
        "exec": "./bin/bitcoind"
      }
    }
  }
}
把datadir和exec改成你自己的路径，分别是数据目录和分叉币的程序路径，最好用绝对路径
如果datadir换了别的路径，请安装datadir下的bitcoin.conf配置，如果没换路径，就删掉datadir下刚才运行时的数据以免冲突，除了bitcoin.conf都删掉即可。
```
5. bitcoin.conf
```
基本不用改，分叉币加上自己的rpc端口配置(rpcport=12345)，如果有改过。
```
6. 修改network.js
```
vi node_modules/bitcore-lib/lib/networks.js
找到下面这段配置
    129 addNetwork({
    130   name: 'livenet',
    131   alias: 'mainnet',
    132   pubkeyhash: 0x00,
    133   privatekey: 0x80,
    134   scripthash: 0x05,
    135   xpubkey: 0x0488b21e,
    136   xprivkey: 0x0488ade4,
    137   networkMagic: 0xbddeb4d9,
    138   port: 7117,
    139   dnsSeeds: [
    140     'seed1.dns.btcd.io',
    141     'seed2.dns.btcd.io',
    142     'seed3.dns.btcd.io',
    143     'seed4.dns.btcd.io',
    144     'seed5.dns.btcd.io',
    145     'seed6.dns.btcd.io'
    146   ]
    147 });
改成分叉币对应的配置。
```
7. 运行
```
bitcored
```

参考链接  
[bitcore](https://bitcore.io/guides/full-node)  
[搭建区块链浏览器](https://www.tiny-calf.com/2017/07/20/%E6%90%AD%E5%BB%BA%E5%8C%BA%E5%9D%97%E9%93%BE%E6%B5%8F%E8%A7%88%E5%99%A8/)  
第一个官方文档不会告诉你各种坑。
