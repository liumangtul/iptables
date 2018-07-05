# 防火墙 -- IPTABLES
## 概述
定义：iptables 是用在linux上的防火墙


作用：管控要进来的，要出去的，或要转发的数据包

用途：
    规则rules


    可以设置特性，去描述数据包


    比如：


        传输的协议，类型


        数据包的来源，去向，或者使用的端口号等


        如果符合设置的规则，将执行里面设置的动作target


    target ：accept接受，drop丢弃，reject拒绝

## Chain 规则的分组 ##
    1. INPUT   接收到的数据包规则
    2. OUTPUT  发出的数据包规则
    3. FORWARD 转发的数据包规则

## rules 规则 ##
    按顺序逐条检查，如果符合某一条rule的描述，就会执行对应rule的target，后面的rules就不会执行了

    设计时，通用的规则放前面，越具体越放后面

## 传输协议 protocol ##
    
    定义：传递动作用的方法（tcp协议等）
#### 起步
    port  端口
    ssh:22
    website:80
    nginx管理工具
    sudo yum install epel-release
    sudo yum install nginx -y


#### 启动nginx
    sudo systemctl start nginx
                    status,stop

#### nmap

    作用：扫描服务器开放的端口
    sudo yum install nmap -y

    nmap -sT  192.168.33.160
       s： 扫描 T：tcp

#### example1 ####
     
    [vagrant@localhost /]$ nmap -sT 192.168.33.10
        Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-04 15:18 UTC
        Nmap scan report for bogon (192.168.33.10)
        Host is up (0.00072s latency).
        Not shown: 997 closed ports
        PORT    STATE SERVICE
        22/tcp  open  ssh
        80/tcp  open  http
        111/tcp open  rpcbind
        Nmap done: 1 IP address (1 host up) scanned in 0.10 seconds
    
    

---
  #### example2 ####  
   
    [vagrant@localhost /]$ nmap -sT localhost
        Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-04 15:26 UTC
        Nmap scan report for localhost (127.0.0.1)
        Host is up (0.00045s latency).
        Other addresses for localhost (not scanned): 127.0.0.1
        Not shown: 996 closed ports
        PORT    STATE SERVICE
        22/tcp  open  ssh
        25/tcp  open  smtp
        80/tcp  open  http
        111/tcp open  rpcbind
        Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
    

### 查看端口上的连接 ###
cat /etc/services
service name 服务名称

port/protocol 端口号/传输协议

aliases 别名

comment 描述

sudo cat /etc/services | grep 80 筛选关键词
----
运行netstat 命令需要先安装 net-tools

sudo yum install net-tools
----

查找哪个命令打开的某个端口
sudo netstat -anp | grep 80
    
    a all
    
    n 使用数字表示的方式

    p 显示进程的id或者名字

```
[vagrant@localhost /]$ sudo netstat -anp | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2906/nginx: master
                        `                   more... ...
```

----
只查找当前使用tcp协议传输的连接
sudo netstat -ntp
```
proto 协议 | Recv-Q | Send-Q | Local Address 本机地址 | Foreign Address 连接服务的主机 | state 状态| PID/Program name
state  ： ESTABLISHED 已经创建

tcp | 0 | 0 | x.x.x.x:xxx | x.x.x.x.:xx | ESTABLISHED | PID、系统名称
```

#### 基础命令 ####
查看现有的防火墙
```
sudo iptables -L --line-numbers
sudo iptables --list
```
备注：policy 默认的规则|政策 ACCEPT 通过
---
查找现有的规则是用什么样的iptables生成的
```
sudo iptables -S
sudo iptables --list-rules

-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT

```

修改默认的规则
```
sudo iptables -P FORWARD DROP
```
备注：把-P policy（默认规则） 里的FORWARD（转发） 执行的规则改成 DROP（丢弃）  

---

#### 禁止指定来源ip的访问 ####
在防火墙应用里指定规则应用的Chain（规则分组），描述数据包应用的样子，执行的动作target
```
    sudo iptables -A INPUT -s 192.168.1.1 -j DROP
    -A append 追加
    INPOUT 进入到服务器的数据 
    -s source 数据包的来源

    追加(append) 丢弃(DROP) 192.168.1.1 进入到服务器(INPUT)

    -A INPUT 规则放在哪

    -s 1.1.1.1 数据包来源
    备注：192.168.1.0/24 从0-24区间的ip -p tcp 指定传输的协议 --dport 目标端口 80

    -j DROP 执行的target动作

    --dport 目标端口指在哪个端口传输协议
```

#### example ####
```
    sudo iptables -A INPUT -s 192.168.1.1/24 -p tcp --dport 80 -j DROP
```

---
### 管理规则 ###

查看

sudo iptables -L --line-numbers

sudo iptables --line

sudo iptables -L INPUT -v 数据量

sudo iptables -Z 清除数据量

查看rules

sudo iptables -S

sudo iptables --list-rules

---

追加，插入rules

按顺序遇到符合的规则就会执行对应的动作，后面就不匹配该规则了

顺序很重要

-A 追加的最后

-I 追加的指定的位置上


sudo iptables -I 放到INPUT上 行号1 -i 
 
 -i lo 允许本地流量 
 
 target 通过

sudo iptables -I INPUT 1 -i lo -j ACCEPT

------
添加一个丢弃所有udp输入的协议规则

sudo iptables -A INPUT -p udp -j DROP

sudo iptables -L --line-numbers

### 删除 ###

删除指定

sudo  iptables -D INPUT 行号

清空规则

                -F INPUT

-----

### 保存 ###

#### 安装管理 ####
sudo yum install iptables-services -y

#### 开启 ####
sudo systemctl start iptables 

#### 开机自启 ####
sudo systemctl enable iptables 

#### 查看 ####
sudo cat /etc/sysconfig/iptables 

#### 重启iptables ####
sudo systemctl restart iptables 

#### 显示规则 ####
sudo iptables -L 

#### 保存iptables ####
sudo service iptables save
sudo systemctl resstart iptables 重启生效

-------

. 每个chain都有一个默认的policy

### 修改policy ###

把默认的policy 从 ACCEPT 改成 DROP

也就是只允许设置的通过，其他未知的DROP丢弃。

sudo iptables -L

sudo iptables -F 清洗掉chain

sudo service iptables save 保存

sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

-m conntrack --ctsate 使用conntrack模块提供的ctstate功能

ESTABLISHED 允许已经创建的连接

RELATED 允许跟现在的连接相关的数据包

-j ACCEPT 允许前面的匹配的动作是 ACCEPT-通过


-----
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

允许端口：22 在tcp协议下传输数据--22 是ssh服务默认使用的端口号

----
sudo iptables -P INPUT DROP

把INPUT的默认policy 设置成DROP，其他的阻止

设置完保存

sudo service iptables save


-----
#### 允许本地流量loopback ####
sudo iptables -I INPUT 1 -i lo -j ACCEPT

-i interface 指定网卡

lo 本地环路设备

----

#### 允许web服务 ####
默认是80,443
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A input -p tcp --dprot 443 -j ACCEPT
sudo service iptables save
sudo systemctl restart iptables



