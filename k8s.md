k8s

### 概念

###### Node

节点，对应一个实体机或是一个虚拟机，一个节点中可以多个Pod。

![img](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

###### Pod

Pod 是 Kubernetes 平台上的原子单元。当我们在 Kubernetes 上创建 Deployment 时， 该 Deployment 会在其中创建包含容器的 Pod（而不是直接创建容器）。一个Pod中可能包含多个容器，卷。Pod 中的容器共享网络命名空间和 IP 地址，它们可以通过 localhost 相互通信。

![img](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

###### service

pod无法在集群外访问，而Service 是一种抽象，用于定义一组 Pod 的访问方式， 为一组 Pod 提供流量路由。Service 提供了一个固定的 IP 地址和端口，作为对一组 Pod 的入口点。通过 Service，可以实现负载均衡、服务发现等功能。当 Service 接收到请求时，它会将请求转发给后端的一组 Pod，这些 Pod 可能位于集群的不同节点上。类似spring cloud中的负载均衡，通过一个统一的service-aaa可访问所有的aaa服务



### kubectl

**`kubectl get [pods|deplotments|services]`**

**`kubectl describe [pods|deplotments|services]`**