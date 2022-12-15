## IPIP 模式

#### 部署两个pod分别在不同节点
```
-> % kubectl get po -owide
NAME   READY   STATUS   IP               NODE      
pod1   1/1     Running  10.244.120.69    nod1
pod2   1/1     Running  10.244.205.195   node2

-> % kubectl get no -owide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   
minikube       Ready    control-plane   17h   v1.24.3   192.168.49.2  
minikube-m02   Ready    <none>          17h   v1.24.3   192.168.49.3   
```

#### node1 路由
<img width="645" alt="image" src="https://user-images.githubusercontent.com/67354376/207768509-1a4f4989-5c3f-48f0-9732-b1b63583f7ae.png">

#### ping 包 之旅
```
pod1 --> pod2 (10.244.120.69 --> 10.244.205.195)
```

在node1上抓包

<img width="676" alt="image" src="https://user-images.githubusercontent.com/67354376/207768153-cd2c5268-962f-44c7-987d-3f22d23b3361.png">

在 node2 上抓包

<img width="708" alt="image" src="https://user-images.githubusercontent.com/67354376/207768263-cd69e13e-cd73-4df3-ab0b-35e606d03f76.png">

抓包 pod2 网卡

<img width="661" alt="image" src="https://user-images.githubusercontent.com/67354376/207768339-d4f0972b-1d4d-49d7-b77e-d83950b96b4c.png">


#### IPIP 连接方式
![image](https://user-images.githubusercontent.com/67354376/207802420-e6a2da57-2365-43bf-899a-639e3f83dac0.png)



## BGP 模式

#### 修改配置
在安装calico网络时，默认安装是IPIP网络。calico.yaml文件中，将CALICO_IPV4POOL_IPIP的值修改成 "off"，就能够替换成BGP网络。
![image](https://user-images.githubusercontent.com/67354376/207819579-3ac525b0-958a-45ac-bd3e-e3d5c49ac586.png)

 

#### 对比
BGP网络相比较IPIP网络，最大的不同之处就是没有了隧道设备 tunl0。 前面介绍过IPIP网络pod之间的流量发送tunl0，然后tunl0发送对端设备。BGP网络中，pod之间的流量直接从网卡发送目的地，减少了tunl0这个环节。

 

master节点上路由信息。从路由信息来看，没有tunl0设备。 

![image](https://user-images.githubusercontent.com/67354376/207819694-4109a8a0-b93e-4fb5-a6ab-a856240b40fc.png)


 

同样创建一个daemonset，pod1在master节点上，pod2在node节点上。

![image](https://user-images.githubusercontent.com/67354376/207819726-de402519-334c-4995-81c9-84f7a3a1a895.png)


 
#### pod1 ping pod2。

![image](https://user-images.githubusercontent.com/67354376/207819846-799ab1db-e8b9-415d-b8c7-19cf40ecb0c9.png)


根据pod1中的路由信息，ping包通过eth0网卡发送到master节点上。

master节点上路由信息。根据匹配到的 192.168.190.192 路由，该路由的意思是：去往网段192.168.190.192/26 的数据包，发送网段172.171.5.96。而5.96就是node节点。所以，该数据包直接发送了5.96节点。 

 ![image](https://user-images.githubusercontent.com/67354376/207819892-9e4ec0a9-26cb-4f22-ad2d-46327a9f4175.png)


 

node节点上的路由信息。根据匹配到的192.168.190.192的路由，数据将发送给 cali6fcd7d1702e设备，该设备和上面分析的是一样，为pod2的veth pair 的一端。数据就直接发送给pod2的网卡。

 ![image](https://user-images.githubusercontent.com/67354376/207820028-86b1b03f-31c2-4fbc-abf8-ac5427c01ab7.png)


 

当pod2对ping包做出回应之后，数据到达node节点上，匹配到192.168.236.0的路由，该路由说的是：去往网段192.168.236.0/26 的数据，发送给网关 172.171.5.95。数据包就直接通过网卡ens160，发送到master节点上。
![image](https://user-images.githubusercontent.com/67354376/207820089-225bc2c9-41fa-4524-bbcd-48f087b7a1d0.png)



 

通过在master节点上抓包，查看经过的流量，筛选出ICMP，找到pod1 ping pod2的数据包。
![image](https://user-images.githubusercontent.com/67354376/207820143-e294f561-c9b0-4abc-9240-313add5701a4.png)



BGP的连接方式：

 ![image](https://user-images.githubusercontent.com/67354376/207820429-27536d24-dd92-4ea5-8841-f8c173336e6d.png)


 


IPIP网络：

流量：tunlo设备封装数据，形成隧道，承载流量。

适用网络类型：适用于互相访问的pod不在同一个网段中，跨网段访问的场景。外层封装的ip能够解决跨网段的路由问题。

效率：流量需要tunl0设备封装，效率略低

BGP网络：

流量：使用路由信息导向流量

适用网络类型：适用于互相访问的pod在同一个网段，适用于大型网络。

效率：原生hostGW，效率高

 


(1) 缺点租户隔离问题

Calico 的三层方案是直接在 host 上进行路由寻址，那么对于多租户如果使用同一个 CIDR 网络就面临着地址冲突的问题。

 

(2) 路由规模问题

通过路由规则可以看出，路由规模和 pod 分布有关，如果 pod离散分布在 host 集群中，势必会产生较多的路由项。

 

(3) iptables 规则规模问题

1台 Host 上可能虚拟化十几或几十个容器实例，过多的 iptables 规则造成复杂性和不可调试性，同时也存在性能损耗。

 

(4) 跨子网时的网关路由问题

当对端网络不为二层可达时，需要通过三层路由机时，需要网关支持自定义路由配置，即 pod 的目的地址为本网段的网关地址，再由网关进行跨三层转发。
