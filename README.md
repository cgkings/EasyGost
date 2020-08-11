# GOST一键脚本使用指南
***
#### 感谢: 
1. 感谢 @ginuerzh 大佬提供了如此优秀实用的软件，项目地址 `https://github.com/ginuerzh/gost` ，GOST功能强大使用简单，想要详细了解用法的朋友可以查看官方文档 `https://docs.ginuerzh.xyz/gost/`  
2. 感谢 @风萧萧兮易水寒 大佬的原始脚本，本脚本的灵感与思路都来自此大佬的一键脚本，项目地址 `https://www.fiisi.com/?p=125` 
***
> 脚本简介
> > 适合懒人的超简单gost一键转发脚本:  

```
wget https://raw.githubusercontent.com/cgkings/EasyGost/master/gost.sh && chmod +x gost.sh && ./gost.sh

```

> > 脚本由 @stsdustjc 制作，参考了 @风萧萧兮易水寒 大佬的一键脚本，主要实现了systemd及gost配置文件对gost进行管理，能够在不借助其他工具(如screen)的情况下实现多条转发规则同时生效。但功能性较弱，目前仅支持tcp+udp不加密转发, relay+tls加密转发, relay+tls解密对接转发。也算是能勉强保护一下裸奔的流量和搭建简单隧道的需求了。  
> > 项目地址及帮助文档:  
> > https://github.com/stsdust/EasyGost  
***
## 本一键脚本适用人群  
* 你只是想赶快用上gost进行流量转发，不想学习gost的详细配置方法，只想越简单越好  
* 你对转发的要求并不高，纯转发或简单加密就能满足要求了  
* 在满足上一条的情况下，你可能有多项gost转发规则，想要管理起来方便些  
##### 如果你不符合上述条件请自行阅读gost的官方文档，gost能够通过简单的命令实现非常多强大的功能，但本脚本并未加入那些功能  
##### 如果你是萌新，有很多与IP/端口/转发有关的概念都还没弄清楚，请跟随本文的「完整食用教程」一步步进行设置，最终通过一键脚本实现转发  

***  
## 完整食用教程  
* 启动脚本: 在已经运行过如下一键脚本的情况下  

```
wget https://raw.githubusercontent.com/cgkings/EasyGost/master/gost.sh && chmod +x gost.sh && ./gost.sh

```
* 再次运行本脚本只需要输入`./gost.sh`回车即可  

***

* 难点1: 协议选择  
![1.png](https://i.loli.net/2020/05/19/xtinMIfkzGcUKZ9.png)  
    * 对于具备足够加密强度的流量，如某些服务(Caddy, Nginx)提供的tls加密，这种情况下你可能只需要对流量进行中转而不需要加密，那么选[1]即可，在这种情况下，你只需要配置一台主机(中转流量的主机)即可实现GOST转发流量  
    * 有一些服务的加密强度是不可靠的，所以你希望通过GOST实现在中转流量的同时对流量进行加密，这种情况下选[2]即可。值得说明的是：  
        - 当选择[2]对中转流量进行加密时，意味着你还需要配置你的远程主机，即接收这些转发流量的主机，需要登录上远程主机运行此脚本，并选择协议[3]对来自于此主机的加密流量进行解密，从而供远程主机的其他服务接收这些流量  
        - 目前本脚本仅采用GOST提供的默认证书，亦可能受到HTTPS中间人攻击，如果想要更加安全，请放弃使用此脚本，自行学习GOST官方文档  
    * 协议[3]用于接收通过[2]转发而来的加密流量，即如果中转主机使用了协议[2]，则要在对应的目标主机上选择协议[3]以对接  

***

* 难点2: 本地端口填写  
![2.png](https://i.loli.net/2020/05/19/kcmiI3j9vRw1brz.png)  
![3.png](https://i.loli.net/2020/05/19/Yv95DEt8yIQXWzx.png)  
    * 本地端口是指，当前正在使用脚本的这台主机要监听的端口，从这个端口注入的流量会经由GOST服务进行处理(转发/加密/解密)  
    * 一般而言，如果使用国内的NAT类机型，分为外网和内网两种端口，这里应填内网端口(如图60001)  
    * 而外部的其他客户端想要连接到这一台NAT主机时则需要在外部的客户端上填写外网端口(37096)以实现端口映射和连接，当然这一部分与本脚本无关，外部端口也无需在本脚本中填写  
    * 一般而言，对于有独立IP的VPS/VDS/独服等类型的主机，端口通常都是全开放的，这里的端口直接自己按需填写即可，不存在内外之分(建议填写10000号以上的端口)  

***

* 难点3: IP填写  
    - IP填写分为几种情况，这里按3种协议选项进行解释  
    - [A]---[B]---[C]，假如A是你自己正在用的电脑，[B]是用于转发流量的主机，[C]是最终接收由[A]发出流量的主机  
    1. 选择协议[1]的，即不加密的流量转发，则此处填写你[C]主机的IP即可  
    2. 在[B]主机上选择协议[2]的，即对流量进行加密，则此处也是填写[C]主机的IP，如图:  
![4.png](https://i.loli.net/2020/05/19/dIOhC1q6fzjmylc.png)  
    3. 在[C]主机上选择协议[3]的，即对由[B]转发过来的流量进行解密，则此处填写[C]主机的IP或`127.0.0.1`，这取决于在[C]主机上，GOST接收到这些流量后要将流量交给[C]主机面向外部的端口，还是面向[C]主机内部本地回环的端口  
        * 一般来讲，只有[A]---[C]的情况下，[C]主机某服务的端口是监听在本机上而非本地回环的，某些服务配置文件一般是这种形式:  

```javascript
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":22332,
```

* 或者这种形式  

```javascript
{
  "inbounds": [
    {
      "port": 22332,
      "protocol": "?????",    
      "settings": {
        "clients": [
```

* 以上2种形式的服务端配置文件都表明服务监听在本机的22332端口上，而不是本地回环，这种情况IP填写本机[C]的公网IP即可  
* 同样的，通过修改你[C]主机上对应服务的配置文件，可以实现让这一服务仅监听在[C]主机本地回环的端口上，如`127.0.0.1:22332`即如下配置所示  

```javascript
    "server":127.0.0.1",
    "local_address":"127.0.0.1",
    "local_port":22332,
```

* 或者这种形式  

```javascript
{
  "inbounds": [
    {
      "port": 22332,
      "listen":"127.0.0.1",
      "protocol": "?????",    
      "settings": {
        "clients": [
```

* 以上2种形式的服务端配置文件都表明服务监听在本地回环(127.0.0.1)的22332端口上，换言之，除了本机内部的流量外，其他来源的流量都无法访问这一服务，因此安全系数更高，而GOST恰恰可以把从外部接收来的流量转发至本机内部  
* 这种情况IP填写为`127.0.0.1`即可  

***

* 难点4: 目标端口填写  
    - 算不上难点，稍微需要注意的地方提示一下  
![5.png](https://i.loli.net/2020/05/19/GsoZh2bwgFeITCX.png)  
    1. 选择协议[1]的，此处填写你[C]主机对应服务监听的端口即可，可能是`443`也可能是如上示例的`22332`，具体依据目标主机对应服务的配置  
    2. 在[B]主机上选择协议[2]的，则此处建议填写`443`，当然你硬要填写其他的我也拦不住  
    3. 在[C]主机上选择协议[3]的，需要比较注意，在[C]主机上选择协议[3]意味着你[B]主机选的是协议[2]，那么[C]主机在用此脚本时第一次填写的本地端口(难点2)，应该与[B]主机上配置时填写的目标端口相一致(难点4.2)，一般是`443`，而第二次填写的端口应与[C]主机上具体服务所监听的端口相一致，按如上示例即为`22332`  

### 到此一切设置完毕，尽情享受GOST为你带来的便捷吧 :)  
