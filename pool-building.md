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
git checkout 7cf1f76cf9e4e52ef3a8c574024469db095fe591
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
2. 安装 node 版本 (当前默认是v8.11.2)
```
nvm install v8.11.2 && nvm use v8.11.2
```
3. 安装web服务器`http-server`和矿池服务持久运行工具`pm2`
```
npm install -g http-server
npm install -g pm2
```
