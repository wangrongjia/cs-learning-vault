## 一、工作负载资源（Workload Resources）

### 1. **Pod**

- **定义**：K8s 中最小的可部署单元，包含一个或多个容器
- **特点**：
    - Pod 内的容器共享网络命名空间（同一个 IP）
    - 共享存储卷
    - 通常一个 Pod 运行一个主容器，可配合 sidecar 容器
- **生命周期**：短暂的，Pod 重启后 IP 会变化
- **使用场景**：一般不直接创建，而是通过控制器管理

### 2. **Deployment**

- **定义**：用于管理无状态应用的控制器
- **核心功能**：
    - 声明式更新 Pod 和 ReplicaSet
    - 滚动更新（Rolling Update）
    - 回滚到之前的版本
    - 扩缩容
- **适用场景**：Web 服务、API 服务等无状态应用
- **示例**：部署 3 个副本的 Nginx

### 3. **StatefulSet**

- **定义**：管理有状态应用的控制器
- **核心特性**：
    - 稳定的网络标识（Pod 名称固定，如 web-0, web-1）
    - 稳定的持久化存储
    - 有序的部署和扩展
    - 有序的删除和终止
    - 每个 Pod 可以挂载独占的持久化存储
- **适用场景**：数据库（MySQL、MongoDB）、消息队列（Kafka）、Zookeeper 集群
- **与 Deployment 区别**：Pod 名称和存储卷绑定关系不变

### 4. **DaemonSet**

- **定义**：确保每个节点上运行一个 Pod 副本
- **特点**：
    - 新节点加入时自动部署 Pod
    - 节点移除时自动删除 Pod
- **适用场景**：
    - 日志收集（Fluentd、Logstash）
    - 监控代理（Prometheus Node Exporter）
    - 网络插件（Calico、Flannel）
    - 存储插件

### 5. **Job**

- **定义**：运行一次性任务的控制器
- **特点**：
    - 保证指定数量的 Pod 成功完成
    - 任务完成后 Pod 不会被删除（便于查看日志）
- **配置选项**：
    - `completions`：需要成功完成的 Pod 数量
    - `parallelism`：并行运行的 Pod 数量
- **适用场景**：批处理、数据迁移、备份任务

### 6. **CronJob**

- **定义**：按照时间调度运行 Job
- **特点**：
    - 使用 Cron 表达式定义调度时间
    - 基于 Job 实现
- **适用场景**：定期备份、报表生成、定时清理任务
- **示例**：每天凌晨 2 点执行数据备份

## 二、服务发现与负载均衡

### 7. **Service**

- **定义**：为一组 Pod 提供稳定的网络访问入口
- **类型**：
    - **ClusterIP**（默认）：集群内部访问，分配虚拟 IP
    - **NodePort**：通过节点 IP + 端口对外暴露（30000-32767）
    - **LoadBalancer**：云厂商提供的负载均衡器
    - **ExternalName**：返回外部服务的 CNAME
- **功能**：
    - 服务发现（通过 DNS）
    - 负载均衡
    - 抽象 Pod IP 的变化

### 8. **Ingress**

- **定义**：管理外部访问集群内服务的 HTTP/HTTPS 路由规则
- **功能**：
    - 基于域名/路径的路由
    - SSL/TLS 终止
    - 负载均衡
- **组成**：Ingress 资源 + Ingress Controller（如 Nginx Ingress）
- **示例**：`api.example.com` 路由到服务 A，`web.example.com` 路由到服务 B

### 9. **Endpoint**

- **定义**：Service 对应的后端 Pod IP 和端口列表
- **工作原理**：
    - Service 通过 Label Selector 选择 Pod
    - K8s 自动创建/更新 Endpoint 对象
    - kube-proxy 根据 Endpoint 配置负载均衡规则
- **手动创建**：可创建无 Selector 的 Service + 手动 Endpoint（用于外部服务）

### 10. **NetworkPolicy**

- **定义**：Pod 之间的网络访问控制策略（防火墙规则）
- **功能**：
    - 控制入站流量（Ingress）
    - 控制出站流量（Egress）
    - 基于 Pod 标签、命名空间、IP 块进行过滤
- **前提**：需要网络插件支持（如 Calico、Cilium）
- **示例**：只允许前端 Pod 访问后端 Pod 的 8080 端口

## 三、配置与存储

### 11. **ConfigMap**

- **定义**：存储非敏感的配置数据（键值对）
- **使用方式**：
    - 环境变量注入
    - 命令行参数
    - 挂载为配置文件
- **特点**：明文存储，可热更新（挂载方式）
- **适用场景**：应用配置、nginx.conf、应用参数

### 12. **Secret**

- **定义**：存储敏感数据（密码、密钥、证书）
- **类型**：
    - Opaque：通用类型（Base64 编码）
    - kubernetes.io/dockerconfigjson：镜像仓库认证
    - kubernetes.io/tls：TLS 证书
- **使用方式**：同 ConfigMap
- **注意**：Base64 编码≠加密，需结合 RBAC 和加密存储

### 13. **Volume**

- **定义**：Pod 中容器可访问的目录
- **常见类型**：
    - **emptyDir**：临时目录，Pod 删除后数据丢失
    - **hostPath**：挂载宿主机目录
    - **configMap/secret**：挂载配置
    - **persistentVolumeClaim**：使用 PV
    - **nfs、cephfs**：网络存储
- **生命周期**：与 Pod 绑定（除 PV 外）

### 14. **PersistentVolume (PV) & PersistentVolumeClaim (PVC)**

- **PV（存储资源）**：
    - 集群级别的存储资源
    - 由管理员创建或动态供应
    - 独立于 Pod 生命周期
- **PVC（存储请求）**：
    - 用户对存储的申请
    - 指定容量、访问模式
- **工作流程**：
    1. 管理员创建 PV 或配置 StorageClass
    2. 用户创建 PVC
    3. K8s 绑定 PVC 到合适的 PV
    4. Pod 通过 PVC 使用存储
- **访问模式**：
    - ReadWriteOnce (RWO)：单节点读写
    - ReadOnlyMany (ROX)：多节点只读
    - ReadWriteMany (RWX)：多节点读写

## 四、集群资源与调度

### 15. **Node**

- **定义**：集群中的工作节点（物理机或虚拟机）
- **组件**：
    - **kubelet**：负责 Pod 生命周期管理
    - **kube-proxy**：负责网络规则
    - **容器运行时**（Docker、containerd）
- **状态**：Ready、NotReady、Unknown
- **资源**：CPU、内存、Pod 数量上限
- **污点（Taint）**：阻止 Pod 调度到该节点

### 16. **Scheduler (kube-scheduler)**

- **定义**：负责将新创建的 Pod 分配到合适的 Node
- **调度流程**：
    1. **预选（Predicate）**：过滤不满足条件的节点
        - 资源是否充足
        - 端口是否冲突
        - 节点选择器匹配
    2. **优选（Priority）**：对候选节点打分
        - 资源均衡
        - 亲和性/反亲和性
        - 选择得分最高的节点
- **调度策略**：
    - 节点亲和性（NodeAffinity）
    - Pod 亲和性/反亲和性
    - 污点和容忍度（Taints & Tolerations）

### 17. **Controller (控制器)**

- **定义**：维护集群期望状态的控制循环
- **工作原理**：
    - 监控资源的当前状态
    - 与期望状态对比
    - 执行操作使其一致
- **常见控制器**：
    - **Deployment Controller**：管理 Deployment
    - **ReplicaSet Controller**：确保副本数量
    - **StatefulSet Controller**：管理有状态应用
    - **DaemonSet Controller**：确保每节点运行 Pod
    - **Job Controller**：管理批处理任务
    - **Node Controller**：监控节点健康
    - **Endpoint Controller**：管理 Endpoint

### 18. **ReplicaSet**

- **定义**：确保指定数量的 Pod 副本运行
- **功能**：
    - 监控 Pod 数量
    - 自动创建/删除 Pod 以达到期望副本数
- **与 Deployment 关系**：
    - Deployment 管理 ReplicaSet
    - Deployment 滚动更新时创建新 ReplicaSet
    - 通常不直接创建 ReplicaSet
- **Label Selector**：通过标签选择管理哪些 Pod

START
Basic
## 五、k8s核心概念关系图
Back:
```
集群层面：
Master Node
├── API Server (入口)
├── Scheduler (调度器)
├── Controller Manager (控制器集合)
└── etcd (存储)

工作节点：
Worker Node
├── kubelet (Pod 管理)
├── kube-proxy (网络代理)
└── Container Runtime

工作负载层次：
Deployment → ReplicaSet → Pod → Container
StatefulSet → Pod (有序) → Container
DaemonSet → Pod (每节点一个)
Job/CronJob → Pod (任务型)

网络访问链路：
外部流量 → Ingress → Service → Endpoint → Pod
```
<!--ID: 1761205772402-->
END
## 六、最佳实践建议

1. **资源管理**：为 Pod 设置 requests 和 limits
2. **健康检查**：配置 livenessProbe 和 readinessProbe
3. **配置分离**：使用 ConfigMap/Secret 管理配置
4. **标签规范**：合理使用 Label 进行资源组织
5. **命名空间**：使用 Namespace 隔离不同环境/项目
6. **版本控制**：所有 YAML 文件纳入 Git 管理
7. **监控告警**：部署 Prometheus + Grafana
8. **日志收集**：使用 EFK/ELK 栈

这些概念构成了 Kubernetes 的核心体系，实际使用中它们相互配合，形成完整的容器编排解决方案。建议从 Pod、Deployment、Service 这些最基础的概念开始实践，逐步深入理解其他高级特性。



