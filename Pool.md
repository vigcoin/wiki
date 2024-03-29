# VIGCOIN 矿池搭建

## 默认环境Ubuntu 20.04


### 安装构建环境
```
sudo apt update && sudo apt install -y build-essential curl cmake libboost-all-dev libssl-dev libsodium-dev
```

### 构建vigcoin程序


1. 下载代码
```
mkdir -p vigcoin
cd vigcoin
git clone https://github.com/vigcoin/coin.git
```
2. 构建程序
```
cd coin
git checkout pool
./build-release.sh
```
3. 导出编译结果

```
mv coin/build/src/ app
```
### 下载矿池服务
```
git clone https://github.com/vigcoin/pool.git pool-server
```

### 下载矿池网页
```
git clone https://github.com/vigcoin/pool-site.git pool-frontend
```

### 搭建node环境

1. 安装nvm环境
```
curl -o- https://raw.githubusercontent.com/calidion/chinese-noder-the-easy-way/master/install.sh?ver=1.1 | bash
export NVM_DIR="$HOME/.nvm" && \. "$NVM_DIR/nvm.sh"
```
2. 安装 node 版本 (默认安装了v12，但是应该无法工作，须安装node.js v8.11.2）
```
nvm install v8.11.2 && nvm use v8.11.2
cd pool-server
npm install
cd ..
```
3. 安装web服务器`http-server`和矿池服务持久运行工具`pm2`
```
npm install -g http-server
npm install -g pm2
```
### 创建钱包（如果已经创建可以忽略)
```
mkdir wallet
../app/simplewallet
```
这时会出现对话框:
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
选择[G]生成新的钱包，并出现如下信息，其中`Generated new wallet`后面的`BHT6WECJdbiEuP37egnYv8GMDTHpMcCNhXxTJsBpvXa715BaMeUyiRRcMdXZqKEGxVG7LEEcMGuXwavZxaK74LZK8v6hrhD`就是你需要保存的钱包地址。
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
创建钱包成功后执行`ls`，会看到`wallet`目录产生了两个文件：
```
mywallet.address  mywallet.wallet
```
请立即备份好这个文件，最好通过zip等压缩软件打包。因为目前服务器代码不够稳定，容易出现钱包被破坏的情况。

### 配置矿池文件
修改配置文件
```
cp pool-server/example-config.json pool-server/config/config.json
vi pool-server/config/config.json
```
修改22行：
```
"poolAddress": "填写你的钱包地址",
```

修改109行:
```
"password": "填写你的后台登录密码"
```
### 配置矿池网站

所在位置：

```
pool-frontend/config/config.json
```

修改第1行:
```
var DOMAIN = "localhost";
```
将localhost修改成你的服务器的域名或者IP

比如修改成是：`pools.vigcoin.org`
```
var DOMAIN = "pools.vigcoin.org";
```
### 配置环境变量
```
# 设置钱包密码
export VIGCOIN_WALLET_PASSWORD=#YOUR_PASSWORD

# 设置服务器代码
export VIGCOIN_DAEMON_PORT=19801
# 设置钱包端口
export VIGCOIN_WALLET_PORT=19802
# 设置其它信息
export VIGCOIN_EXTRA="--enable-cors *"   # 允许远程的RPC
```

### 把脚本放入到crontab启动

1. 获取vigcoin deamon启动脚本命令
```
echo "`pwd`/app/vigcoind --rpc-bind-ip=0.0.0.0  $VIGCOIN_EXTRA"
```
结果效果：
```
/home/vigcoin/vigcoin/app/vigcoind --rpc-bind-ip=0.0.0.0  --enable-cors *
```
2. 获取vigcoin 钱包服务脚本命令
```
echo "`pwd`/app/simplewallet --wallet-file `pwd`/wallet/mywallet.wallet --password $VIGCOIN_WALLET_PASSWORD --daemon-port $VIGCOIN_DAEMON_PORT --rpc-bind-port $VIGCOIN_WALLET_PORT --rpc-bind-ip=0.0.0.0 $VIGCOIN_EXTRA"
```
结果效果：
```
/home/vigcoin/vigcoin/app/simplewallet --wallet-file /home/vigcoin/vigcoin/wallet/mywallet.wallet --password 12345678 --daemon-port 19801 --rpc-bind-port 19802 --rpc-bind-ip=0.0.0.0 --enable-cors
```
3. 获取pool server的启动脚本命令
```
echo "`which node` `which pm2` start `pwd`/pool-server/init.js"
```
结果效果：
```
/home/vigcoin/.nvm/versions/node/v8.11.2/bin/node /home/vigcoin/.nvm/versions/node/v8.11.2/bin/pm2 start /home/vigcoin/vigcoin/pool-server/pool-server/init.js
```
4. 获取启动pool frontend的脚本命令
```
echo "`which node` `which http-server` `pwd`/pool-frontend/"
```
结果效果：
```
/home/vigcoin/.nvm/versions/node/v8.11.2/bin/node /home/vigcoin/.nvm/versions/node/v8.11.2/bin/http-server /home/vigcoin/vigcoin/pool-server/pool-frontend/
```
#### 编辑crontab将结果放入启动脚本
```
crontab -e
```
将下面的内容编辑进去
```
@reboot /home/vigcoin/vigcoin/app/vigcoind --rpc-bind-ip=0.0.0.0  --enable-cors *
@reboot /home/vigcoin/vigcoin/app/simplewallet --wallet-file /home/vigcoin/vigcoin/wallet/mywallet.wallet --password 12345678 --daemon-port 19801 --rpc-bind-port 19802 --rpc-bind-ip=0.0.0.0 --enable-cors
@reboot /home/vigcoin/.nvm/versions/node/v8.11.2/bin/node /home/vigcoin/.nvm/versions/node/v8.11.2/bin/pm2 start /home/vigcoin/vigcoin/pool-server/pool-server/init.js
@reboot /home/vigcoin/.nvm/versions/node/v8.11.2/bin/node /home/vigcoin/.nvm/versions/node/v8.11.2/bin/http-server /home/vigcoin/vigcoin/pool-server/pool-frontend/
```
保存退出。

### 安装redis-server
```
sudo apt install redis-server
```

然后重启服务器。
等数据同步完成即可开启挖矿了。
