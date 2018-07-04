# 防火墙 -- IPTABLES
## 概述
定义：iptables 是一种用在linux上的防火墙


作用：管控要进来的，要出去的，或要转发的数据包


用途：规则rules


    可以设置特性，去描述数据包


    比如：


        传输的协议，类型


        数据包的来源，去向，或者使用的端口号等


        如果符合设置的规则，将执行里面设置的动作target


    target ：accept接受，drop丢弃，reject拒绝

## Chain 规则的分组
    1. INPUT   接收到的数据包规则
    2. OUTPUT  发出的数据包规则
    3. FORWARD 转发的数据包规则

### rules 规则
    按顺序逐条检查，如果符合某一条rule的描述，就会执行对应rule的target，后面的rules就不会执行了

    设计时，通用的规则放前面，越具体越放后面