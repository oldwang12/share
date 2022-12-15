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
