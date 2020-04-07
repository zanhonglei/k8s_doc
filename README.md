---
title: K8S笔记
tags: DevOps
renderNumberedHeading: true
grammar_cjkRuby: true
---
# 知识点
1. Kubernetes 关键字
2. 什么是pod
3. 控制器类型
4. 网络通信模型
5. 构建K8S集群
6. 掌握资源清单语法；
7. 编写pod；
8. 掌握pod生命周期；
9. pod控制器：掌握各种控制器的特点和使用方式；
10. 服务发现：掌握SVC原理，构建方式；
11. 存储：掌握多种存储类型的特点；
12. 调度器：调度器原理，能根据要求在想要的节点上运行；
13. 安全：集群的认证，鉴权，访问控制原理及流程；
14. 服务分类：状态服务：无状态服务 ： LVS，有状态服务：DBMS
15. HELM：掌握HELM原理，HELM模板自定义，HEML部署的一些常用的插件；
16. 运维：修改Kubeadm 证书时效，构建高可用的K8S集群；


# K8S组织架构
- API SERVICE：所有组件的入口，核心
- RC：期望副本数量
- scheduler：任务分配
- etcd：键值对数据库，所有K8S的所有信息（持久化）
- kubelet：直接跟容器交互实现容器生命周期管理；
- kube-proxy：负责写入规则至防火墙（IPTABLES），LVS，实现服务映射访问；
- CoreDNS：创建一个域名解析服务器，解析SVC的域名；
- DashBoard：K8S管理平台；
- ingress controller：官方实现了4层代理，ingress实现了7层代理
- federation：跨多集群中心统一管理功能；
- Prometheus：K8S监控
- ELK：日志采集

# 基础概念
## POD
- 自主式pod
- 被管理的pod


# 资源
K8s 中所有的内容都抽象为资源， 资源实例化之后，叫做对象
## 名称空间级别
- 工作负载型资源( workload ): 
Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、 CronJob ( ReplicationController 在 v1.11 版本被废弃 )

- 服务发现及负载均衡型资源( ServiceDiscovery LoadBalance ): 
Service、Ingress、... 配置与存储型资源: Volume( 存储卷 )、CSI( 容器存储接口,可以扩展各种各样的第三方存储卷 )

- 特殊类型的存储卷:ConfigMap( 当配置中心来使用的资源类型 )、Secret(保存敏感数据)、 DownwardAPI(把外部环境中的信息输出给容器)

- 集群级资源:
Namespace、Node、Role、ClusterRole、RoleBinding、ClusterRoleBinding

- 元数据型资源:
HPA、PodTemplate、LimitRange

## 资源清单
在 k8s 中，一般使用 yaml 格式的文件来创建符合我们预期期望的 pod ，这样的 yaml 文件我们一般 称为资源清单
### 资源清单语法：

|  参数名   | 字段类型    |  说明   |
| --- | --- | --- |
|  version   |  String   | 指的是K8S API 的版本，目前基本上是v1，可以用kubectl api-versions命令查询    |
|   kind  | String    | 指的是YMAL文件定义的资源类型和角色，如Pod ，Service，ReplicaSet   |
| metadata    | Object    | 元数据类型，固定值就写metadata    |
| metadata.name    | String    | 元数据对象名字，这里由我们填写，比如命名Pod的名字    |
| metadata.namespace    | String    | 元数据对象的命名空间，由我们自身定义    |
| Spec    | Object    | 详细定义对象，固定值就写Spec    |







# 什么是控制器
K8S内建了很多的Controller（控制器），用来控制Pod的具体状态和行为；
## 控制器类型
- ReplicationController 和 ReplicaSet
- Deployment
- DaemoSet
- StateFulSet
- Job/CronJob
- Horizontal Pod Autoscaling

## ReplicationController 和 ReplicaSet
RC 用来确保容器应用的副本数始终保持用户定义的副本数
如果容器异常退出，会自动创建新的Pod来替代；
如果异常多出来的容器会被自动回收

在新的K8S版本，建议使用RS代替RC，RS和RC只有一点不同，RS支持集合时的selector：通过标签来统一配置管理；

## Deployment
Deployment 为 pod和 RS提供了声明式定义方法，用来替代之前的RC来方便管理普通应用，应用场景包括：
- 定义Deployment 来创建 Pod和RS
- 滚动升级和回滚应用
- 扩容和缩容
- 暂停和继续Deployment

## DaemonSet
DaemonSet 确保全部或者部分Node上运行一个Pod副本；
当有Node加入集群时，也会为他新建一个Pod；
当有Node从集群移出时，这些Pod也会被回收；
删除DaemonSet将会删除它创建的所有Pod
应用场景包括：
- 运行集群存储DaemonSet,假如在每个Node上运行glusterd，ceph
- 在每个Node上运行日志收集：例如：fluentd，logstash
- 在每个Node上运行监控Daemon，例如：Prometheus Node Exporter ，Collectd，Datadog代理，New Relic代理等

## Job
job负责批量处理任务，即任务执行一次，保证执行批处理任务的一个Pod或者多个成功结束；

## CronJob
Cron job 管理基于时间的job，即
- 在给定的时间只运行一次
- 周期性的运行
典型应用场景：
- 在给定的时间点调度job运行
- 创建周期性运行的job，如：数据库备份，发送邮件；

## StatefulSet
StatefulSet 作为Controller 为Pod提供唯一标识，可以保证部署和Scale的顺序；
StatefulSet 是为了解决有状态服务的问题（对应Deployment和RS是为无状态服务而设计）；
应用场景包括：
- 稳定的持久化存储，即Pod重新调度后还是能访问相同的持久化数据，基于PVC来实现；
- 稳定的网络标识，即Pod重新调度后其PodName和HostName不变，预计Headless Service （即没有ClusterIP的Service）来实现；
- 有序部署，有序口占，即Pod是有顺序的，在部署和扩展的时候要根据定义的顺序依次进行，基于init containers来实现（从0到N，在下一个Pod运行之前所有之前的Pod必须都是running 和 Ready状态）；
- 有序收缩，有序删除

## Horizontal Pod Autoscaling（HPA）
应用的资源使用率通常都有高峰和低谷的时候；
- 削峰填谷，提高集群的整体资源利用率，让Service中的Pod个数自动调整，使Pod水平自动缩放；

  
 # 当pod报错的时候，排查思路：
 1. 多次查看pod信息：error，重启次数： kubectl get pod 
 2. 查看pod的配置信息： kubectl descript pod [pod_name]
 3. 查看对应容器的pod日志 ：kubectl log [pod_name] -c [container-name]



