---
title: AWS+shadowsocks
date: 2018-12-15 20:02:23
tags: 随笔
---

说来惭愧，今天才来折腾翻（he）墙（xie），出来溜达的感觉真好～

## 步骤

- 在这里用到了AWS(Amazon Web Service)，是亚马逊给我们提供的服务，我们可以在`https://amazonaws-china.com/`这里申请服务器。
- 申请成功并将实例启动之后，进入`AWS管理控制台`，选择左上角的`EC2`，再点击`实例`，`连接`，按照官方的指示，在本地连接上服务器。


<!-- more -->

- 然后进行如下操作
```
sudo -s 

apt-get update

apt-get install python-pip


-- 直接install shadowsocks会报错
export LC_ALL=C

pip install shadowsocks
```

- 至此，已经安装好shadowsocks了，下一步就是修改配置文件，然后启动

```
vi /etc/shadowsocks.json
```

输入
```json
{
    "server":"0.0.0.0",
    "server_port":"yourport",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"yourpassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

启动
```
ssserver -c /etc/shadowsocks.json -d start
```

- 将shadowsocks加入自启动
```
vi /etc/rc.local
```

添加
```
ssserver -c /etc/shadowsocks.json -d start
```

- 最后一步，就是在本机下载shadowsocks客户端了，然后配置服务器的公网ip，开放端口以及密码

下载地址`https://github.com/shadowsocks/ShadowsocksX-NG/releases`