# VIGCOIN 矿池搭建

## 默认环境Ubuntu 18.04


### 安装构建环境
```
sudo apt update && sudo apt install -y build-essential curl cmake libboost-all-dev libssl-dev libsodium-dev
```

### 构建vigcoin程序


1. 下载代码
```
git clone https://github.com/vigcoin/coin.git
```
2. 构建程序
```
cd coin
./build-release.sh
```
