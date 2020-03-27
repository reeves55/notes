#SS代理 

## 安装SS

### 1. 下载脚本

```shell
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
```

### 2. 将脚本改为可执行文件

```shell
chmod +x shadowsocks.sh
```

### 3. 运行脚本文件，并输出日志

```shell
./shadowsocks.sh 2>&1|tee shadowsocks.log
```

## 切换内核，启用锐速

### 1. 下载脚本

```shell
wget --no-check-certificate -O rskernel.sh https://raw.githubusercontent.com/uxh/shadowsocks_bash/master/rskernel.sh && bash rskernel.sh
```

### 2. 安装并开启锐速

```shell
yum install net-tools -y && wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && bash appex.sh install
```

