# VIG Coin 节点与本地钱包服务器搭建指南

## 支持环境

docker Ubuntu:16.04

> 其它环境的问题官方暂不作支持。

## 一、初始化目录

1. git获取相关基础文件

```
git clone --depth 1 https://github.com/vigcoin/docker-folder.git
cd docker-folder
```

## 二、安装Docker

```
sudo apt-get install docker.io
sudo usermod -aG docker ${USER}
```
注意第一次添加时需要退出一次shell。

## 三、下载Docker Image `vigcoin/core`并运行

```
docker pull vigcoin/core
docker run -it -v `(pwd)`/wallet:/wallet vigcoin/core
```
正常你会看到提示符的变化：
```
root@50a7f15d5913:/#
```

## 四、通过容器命令创建钱包

1. `cd wallet/`进行vigcoin应用目录
```
root@50a7f15d5913:/# cd wallet/
```
2. 运行`/app/simplewallet`启动钱包创建
```
root@ecec9a07593f:/wallet# /app/simplewallet
vigcoin wallet v1.2.0.1()
Dir: `/root/.vigcoin` not found! Creating...
Success creating dir: /root/.vigcoin.
Creating File: /root/.vigcoin/blockchainindices.dat.
Nor 'generate-new-wallet' neither 'wallet-file' argument was specified.
What do you want to do?
[O]pen existing wallet, [G]enerate new wallet file or [E]xit.
```

3. 输入G表示生成新钱包

```
G
Specify wallet file name (e.g., wallet.bin).
Wallet file name: mywallet    # 输入钱包名   
password: *********           # 输入钱包密码
Error: wallet failed to connect to daemon (http://localhost:19801).
Generated new wallet: BHT6WECJdbiEuP37egnYv8GMDTHpMcCNhXxTJsBpvXa715BaMeUyiRRcMdXZqKEGxVG7LEEcMGuXwavZxaK74LZK8v6hrhD
view key: bbaf1770e78f6f2178e61cd169a3eb79ea3184d33a7b110ae0ed84285a62cf04
**********************************************************************
Your wallet has been generated.
Use "help" command to see the list of available commands.
Always use "exit" command when closing simplewallet to save
current session's state. Otherwise, you will possibly need to synchronize 
your wallet again. Your wallet key is NOT under risk anyway.
**********************************************************************
```

这里`mywallet`不可以修改。 （但是你自己保存时可以随意修改)
这个时候新钱包就创建成功了。

其中：
BHT6WECJdbiEuP37egnYv8GMDTHpMcCNhXxTJsBpvXa715BaMeUyiRRcMdXZqKEGxVG7LEEcMGuXwavZxaK74LZK8v6hrhD
是地址。
同时一定要记住输入的密码。

4. 输入两次`exit`退出docker

效果如下：
```
[wallet BHT6WE]: exit
Wallet closed
root@50a7f15d5913:/app# exit
exit
```

5. 这个时候在wallet目录会有一个`mywallet.wallet`钱包。
这个钱包你也可以从别的地方复制过来。但是要确保名字修改成`mywallet.wallet`。这个时候你可以通过ls命令查看，确保`mywallet.wallet`存在。

```
ls wallet/
mywallet.address  mywallet.wallet
```

看到`mywallet.wallet`表示执行已经成功了。

6. 备份钱包，由于钱包极容易被破坏，所以一定要通过zip打包，然后多备份几份。最好对zip再添加一层密码。


## 五、运行节点与钱包服务容器

1. 设置密码
```
export VIGCOIN_WALLET_PASSWORD=#YOUR_PASSWORD
```

其中#YOUR_PASSWORD就是要被替换的内容。
假设你的密码是`abcd$1234`.
那么应该这样设置：
```
export VIGCOIN_WALLET_PASSWORD="abcd\$1234"
```

2. 设置数据路径
```
export VIGCOIN_HOME=#YOUR_PATH
```
通常如果你没有变更目录，你使用
```
export VIGCOIN_HOME=`(pwd)`
```
即可。

3. 设置端口和默认扩展
```
export VIGCOIN_DAEMON_PORT=19801
export VIGCOIN_WALLET_PORT=19802
export VIGCOIN_EXTRA=""
```

4. 运行节点/钱包服务

```
docker pull vigcoin/core

docker run -d                                                   \
-e VIGCOIN_DAEMON_PORT=$VIGCOIN_DAEMON_PORT                     \
-e VIGCOIN_WALLET_PORT=$VIGCOIN_WALLET_PORT                     \
-e VIGCOIN_EXTRA=$VIGCOIN_EXTRA                                 \
-e PASSWORD=$VIGCOIN_WALLET_PASSWORD                            \
-p 19800:19800                                                  \
-p 19801:19801                                                  \
-p 19802:19802                                                  \
-v $VIGCOIN_HOME/wallet:/wallet                                 \
-v $VIGCOIN_HOME/data:/data                                     \
-v $VIGCOIN_HOME/log:/var/log                                   \
-v $VIGCOIN_HOME/root/.vigcoin:/root/.vigcoin                   \
vigcoin/coind
```

5. 等待同步完成,目前的数据大小约在700M。大约需要几个小时同步。
6. 同步完成后， 通过`docker ps`查看当前docker的容器ID`CONTAINER ID`。
```
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                                                  NAMES
2ef0ee4f04e6        vigcoin/coind        "/usr/bin/supervisord"   8 seconds ago       Up 6 seconds        0.0.0.0:19800-19802->19800-19802/tcp   hungry_jennings
```
7. 登录运行中的docker容器。
```
docker exec -it 2ef0ee4f04e6 /bin/bash
```

8. 通过`netstat -nptl`查找在8119端口侦听的矿池程序
```
root@2ef0ee4f04e6:/pool# netstat -nptl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:19800           0.0.0.0:*               LISTEN      10/vigcoind     
tcp        0      0 0.0.0.0:19801           0.0.0.0:*               LISTEN      10/vigcoind     
tcp        0      0 0.0.0.0:19802           0.0.0.0:*               LISTEN      12/simplewallet   
```

看到上面的两个进程就表示成功了。
如果数据没有同步完时，可能登录后无法操作，这时请耐心等数据同步完成。

9. 自动随服务器重启

在命令行第一行后面添加即可：

```
--restart unless-stopped                                        \
```

即

```
docker run -d --restart unless-stopped                          \
-e VIGCOIN_DAEMON_PORT=$VIGCOIN_DAEMON_PORT                     \
-e VIGCOIN_WALLET_PORT=$VIGCOIN_WALLET_PORT                     \
-e VIGCOIN_EXTRA=$VIGCOIN_EXTRA                                 \
-e PASSWORD=$VIGCOIN_WALLET_PASSWORD                            \
-p 19800:19800                                                  \
-p 19801:19801                                                  \
-p 19802:19802                                                  \
-v $VIGCOIN_HOME/wallet:/wallet                                 \
-v $VIGCOIN_HOME/data:/data                                     \
-v $VIGCOIN_HOME/log:/var/log                                   \
-v $VIGCOIN_HOME/root/.vigcoin:/root/.vigcoin                   \
vigcoin/coind

```

10. 通过`docker ps`查看容器ID， 通过`docker stop 容器ID`可以关闭矿池的运行。

```
docker stop 2ef0ee4f04e6
```
这里注意要使用stop，这样数据才可以安全的保存

11. 

```
docker run -it                                                  \
-e VIGCOIN_DAEMON_PORT=$VIGCOIN_DAEMON_PORT                     \
-e VIGCOIN_WALLET_PORT=$VIGCOIN_WALLET_PORT                     \
-e VIGCOIN_EXTRA=$VIGCOIN_EXTRA                                 \
-e PASSWORD=$VIGCOIN_WALLET_PASSWORD                            \
-p 19800:19800                                                  \
-p 19801:19801                                                  \
-p 19802:19802                                                  \
-v $VIGCOIN_HOME/wallet:/wallet                                 \
-v $VIGCOIN_HOME/data:/data                                     \
-v $VIGCOIN_HOME/log:/var/log                                   \
-v $VIGCOIN_HOME/root/.vigcoin:/root/.vigcoin                   \
vigcoin/coind
```

## 完成

这样你的节点与钱包服务就完成了。

你现在可以对你的钱包进行相关的接口开发了。












