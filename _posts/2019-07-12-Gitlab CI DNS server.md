配置一个DNS服务器，能让其他容器解析到dict.gitlab.com
首先，在gitlab ci服务器上把dict.gitlab.com从/etc/hosts里删除
这时候在gitlab ci服务器上是ping不同dict.gitlab.com的

### 启动DNS服务器
找一台新的linux host,装好docker,大家可以用vagrant来创建一台，然后创建一个dnsmasq的容器,并运行

```
docker run -d -p 53:53/tcp -p 53:53/udp --cap-add=NET_ADMIN --name dns-server andyshinn/dnsmasq
```

### 配置DNS服务
#### 进入容器

```
docker exec -it dns-server /bin/sh
```

1. 首先配置上行的真正的dns服务器地址,毕竟这只是个本地代理,不了解外部规则,创建文件：
```
vim /etc/resolv.dnsmasq
nameserver 114.114.114.114 nameserver 8.8.8.8
```
2. 配置本地解析规则,这才是我们的真正目的.新建配置文件,添加解析规则,其中192.168.211.10是gitlab服务器地址
```
vi /etc/dnsmasqhosts
192.168.211.10 dict.gitlab.com
```
3. 修改dnsmasq配置文件,指定使用上述我们自定义的配置文件
```
vi /etc/dnsmasq.conf
resolv-file=/etc/resolv.dnsmasq addn-hosts=/etc/dnsmasqhosts
```

回到宿主机,重启dns-server容器服务
```
docker restart dns-server
```
这时候这台docker host就是一台DNS服务器了,假如它的地址是192.168.99.100

### 测试
在gitlab ci机器上修改 
```
sudo vim /etc/resolv.conf
nameserver 192.168.99.100
```
- 这时候在本地就可以ping通dict.gitlab.com
- 在gitlab ci创建一个container,进入container后也可以ping通
