 * [1. 高效便捷](#1)
   * [1.1 uniq](#1.1)
 * [2. 网络](#2)
   * [2.1 route](#2.1)
<h2 id="1">高效便捷</h2>

<h3 id="1.1">uniq</h3>
 uniq命令：用于报告或者忽略文件中连续的重复行
 
 * -c：进行计算，并删除文件中重复出现的行
 * -d：仅显示连续的重复行
 * -u：仅显示出现一次的行
```shell
uniq x.txt
```

<h2 id="2">网络</h2>

<h3 id="2.1">route</h3>

```sh
route -n -6(ipv6)
-------------------------------------------------------------------------------------------
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.72.164.1     0.0.0.0         UG    0      0        0 net2
10.72.128.197   0.0.0.0         255.255.255.255 UH    0      0        0 cali349ecc3net0
-------------------------------------------------------------------------------------------
```
* U up表示当前为启动状态
* H host表示该路由为一个主机，多为达到数据包的路由
* G Gateway 表示该路由是一个网关，如果没有说明目的地是直连的
* D Dynamicaly 表示该路由是重定向报文修改
* M 表示该路由已被重定向报文修改
