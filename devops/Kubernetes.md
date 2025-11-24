# Kubernetes 服务参考指南

> 范围：集群管理、资源编排、监控部署  
> Tag 前缀：`[k8s]` `[kubectl]` `[pod]` `[deployment]`  
> 版本：Kubernetes 1.24+  
> 最后更新：2025-11-24

---

## 目录

1. [安装和启动](#安装和启动)
2. [核心概念](#核心概念)
3. [集群管理](#集群管理)
4. [资源管理](#资源管理)
5. [Pod管理](#pod管理)
6. [工作负载](#工作负载)
7. [服务和网络](#服务和网络)
8. [配置和存储](#配置和存储)
9. [实战场景](#实战场景)
10. [监控和调试](#监控和调试)

---

## 安装和启动

### Ubuntu上安装kubectl

```bash
# 方式1：使用官方仓库
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

# 方式2：下载二进制文件
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# 验证安装
kubectl version --client
```

### 安装minikube（本地开发）

```bash
# 安装minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 启动minikube集群
minikube start --driver=docker

# 查看minikube状态
minikube status

# 进入minikube控制台
minikube dashboard
```

### 连接到Kubernetes集群

```bash
# 配置kubeconfig
mkdir -p ~/.kube
# 复制集群提供的kubeconfig文件到~/.kube/config

# 查看当前上下文
kubectl config current-context

# 列出所有上下文
kubectl config get-contexts

# 切换上下文
kubectl config use-context <context-name>

# 查看当前连接
kubectl cluster-info

# 验证连接
kubectl get nodes
```

---

## 核心概念

### 概念 1: Pod（豆荚）

**定义**: Pod是Kubernetes中最小的可部署单元，包含一个或多个容器（通常是Docker容器）。

**特点**:
- 共享网络命名空间（同一IP地址）
- 可共享存储卷
- 生命周期相同（一起启动、停止）
- 通常由控制器管理

**Pod YAML示例**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: sidecar
    image: sidecar:1.0
```

### 概念 2: Deployment（部署）

**定义**: Deployment声明式地管理Pods和ReplicaSets，确保应用程序的指定副本数始终运行。

**关键特性**:
- 副本管理：指定desired replicas数量
- 滚动更新：逐步替换旧Pod，无中断服务
- 回滚：快速回到前一个稳定版本
- 自愈：Pod失败时自动重启

**Deployment YAML示例**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 概念 3: Service（服务）

**定义**: Service是一个抽象，定义了Pod的访问策略和网络端点，提供稳定的访问入口。

**服务类型**:
- ClusterIP：仅在集群内访问（默认）
- NodePort：通过节点IP:端口访问
- LoadBalancer：使用云提供商的负载均衡器
- ExternalName：映射到外部DNS

**Service YAML示例**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 概念 4: ConfigMap和Secret

**ConfigMap**: 存储非机密的配置数据

**Secret**: 存储机密数据（密码、API密钥等）

---

## 集群管理

### 查看集群信息

```bash
# 查看集群信息
kubectl cluster-info

# 查看集群中的节点
kubectl get nodes

# 查看节点详细信息
kubectl describe node <node-name>

# 查看节点资源占用
kubectl top nodes

# 查看API服务器信息
kubectl api-resources

# 查看当前API服务器版本
kubectl api-versions
```

### 节点管理

```bash
# 列出所有节点
kubectl get nodes -o wide

# 查看节点详细信息
kubectl describe node <node-name>

# 标记节点（添加标签）
kubectl label nodes <node-name> tier=backend

# 删除节点标签
kubectl label nodes <node-name> tier-

# 将节点设为不可调度（维护模式）
kubectl cordon <node-name>

# 将节点设为可调度
kubectl uncordon <node-name>

# 优雅驱逐节点上的Pod（用于维护）
kubectl drain <node-name> --ignore-daemonsets

# 查看节点容量和可分配资源
kubectl describe node <node-name> | grep -A 5 "Allocatable"
```

### 集群资源配额

```bash
# 查看所有命名空间
kubectl get namespaces

# 创建命名空间
kubectl create namespace my-app

# 删除命名空间（会删除其中所有资源）
kubectl delete namespace my-app

# 查看资源配额
kubectl describe quota -n my-app

# 创建资源配额（ResourceQuota）
cat > quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: my-app
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "100"
EOF
kubectl apply -f quota.yaml
```

---

## 资源管理

### Deployment管理

```bash
# 列出所有Deployment
kubectl get deployments

# 列出特定命名空间的Deployment
kubectl get deployments -n my-app

# 查看Deployment详细信息
kubectl describe deployment <deployment-name>

# 创建Deployment
kubectl create deployment myapp --image=myapp:1.0 --replicas=3

# 使用YAML文件创建
kubectl apply -f deployment.yaml

# 编辑Deployment
kubectl edit deployment <deployment-name>

# 更新Deployment的镜像
kubectl set image deployment/<deployment-name> myapp=myapp:2.0

# 更新资源限制
kubectl set resources deployment/<deployment-name> -c=myapp --limits=cpu=200m,memory=512Mi

# 查看Deployment的滚动更新历史
kubectl rollout history deployment/<deployment-name>

# 查看特定版本的详细信息
kubectl rollout history deployment/<deployment-name> --revision=2

# 回滚到上一个版本
kubectl rollout undo deployment/<deployment-name>

# 回滚到特定版本
kubectl rollout undo deployment/<deployment-name> --to-revision=2

# 暂停Deployment的更新
kubectl rollout pause deployment/<deployment-name>

# 恢复Deployment的更新
kubectl rollout resume deployment/<deployment-name>

# 查看滚动更新状态
kubectl rollout status deployment/<deployment-name>

# 删除Deployment
kubectl delete deployment <deployment-name>
```

### Pod管理

```bash
# 列出所有Pod
kubectl get pods

# 列出所有Pod（显示更多信息）
kubectl get pods -o wide

# 查看Pod详细信息
kubectl describe pod <pod-name>

# 查看Pod日志
kubectl logs <pod-name>

# 查看容器日志（Pod有多个容器时）
kubectl logs <pod-name> -c <container-name>

# 实时跟踪Pod日志
kubectl logs -f <pod-name>

# 进入Pod容器
kubectl exec -it <pod-name> -- /bin/bash

# 进入特定容器（Pod有多个容器时）
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# 在Pod中执行命令
kubectl exec <pod-name> -- ls -la

# 复制文件从Pod到本地
kubectl cp <pod-name>:/path/in/pod ./local/path

# 复制文件从本地到Pod
kubectl cp ./local/path <pod-name>:/path/in/pod

# 删除Pod
kubectl delete pod <pod-name>

# 强制删除Pod
kubectl delete pod <pod-name> --grace-period=0 --force
```

### ReplicaSet管理

```bash
# 列出所有ReplicaSet
kubectl get replicasets

# 查看ReplicaSet详细信息
kubectl describe replicaset <rs-name>

# 扩展ReplicaSet的副本数
kubectl scale replicaset <rs-name> --replicas=5

# 删除ReplicaSet（会删除Pod）
kubectl delete replicaset <rs-name>

# 删除ReplicaSet但保留Pod
kubectl delete replicaset <rs-name> --cascade=orphan
```

### StatefulSet（有状态应用）

```bash
# 列出所有StatefulSet
kubectl get statefulsets

# 创建StatefulSet
kubectl apply -f statefulset.yaml

# 查看StatefulSet详细信息
kubectl describe statefulset <statefulset-name>

# 删除StatefulSet
kubectl delete statefulset <statefulset-name>
```

---

## Pod管理

### Pod生命周期和健康检查

```bash
# 配置liveness probe（存活性探针）
cat > pod-with-health-check.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30    # 启动后30秒开始探测
      periodSeconds: 10          # 每10秒探测一次
      timeoutSeconds: 5          # 探测超时5秒
      failureThreshold: 3        # 连续失败3次后重启
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 1
EOF
kubectl apply -f pod-with-health-check.yaml
```

### Pod调度

```bash
# 查看Pod被调度到哪个节点
kubectl get pods -o wide

# 使用nodeSelector指定节点
cat > pod-with-nodeselector.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  nodeSelector:
    tier: backend
  containers:
  - name: app
    image: myapp:1.0
EOF

# 使用亲和性规则（更灵活）
cat > pod-with-affinity.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: tier
            operator: In
            values:
            - backend
  containers:
  - name: app
    image: myapp:1.0
EOF

# 污点和容忍（Taints and Tolerations）
kubectl taint nodes <node-name> role=master:NoSchedule
```

---

## 工作负载

### 常见工作负载类型

```bash
# Deployment（无状态应用）
kubectl create deployment myapp --image=myapp:1.0 --replicas=3

# StatefulSet（有状态应用，如数据库）
kubectl apply -f statefulset.yaml

# DaemonSet（每个节点运行一个，如日志收集）
kubectl apply -f daemonset.yaml

# Job（一次性任务）
kubectl apply -f job.yaml

# CronJob（定时任务）
kubectl apply -f cronjob.yaml
```

### Job示例

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
      - name: task
        image: myapp:1.0
        command: ["python", "script.py"]
      restartPolicy: Never
  backoffLimit: 3
```

### CronJob示例

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # 每天凌晨2点执行
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:1.0
            command: ["/backup.sh"]
          restartPolicy: OnFailure
```

---

## 服务和网络

### Service配置

```bash
# 创建ClusterIP Service
kubectl expose deployment myapp --port=80 --target-port=8080

# 创建NodePort Service
kubectl expose deployment myapp --type=NodePort --port=80 --target-port=8080

# 创建LoadBalancer Service（云环境）
kubectl expose deployment myapp --type=LoadBalancer --port=80 --target-port=8080

# 查看Service
kubectl get services

# 查看Service详细信息
kubectl describe service <service-name>

# 删除Service
kubectl delete service <service-name>
```

### Service YAML示例

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
---
# NodePort示例
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30000  # 30000-32767范围内
---
# LoadBalancer示例
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### Ingress（入口）

```bash
# 创建Ingress
kubectl apply -f ingress.yaml

# 查看Ingress
kubectl get ingress

# 查看Ingress详细信息
kubectl describe ingress <ingress-name>
```

### Ingress YAML示例

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

---

## 配置和存储

### ConfigMap

```bash
# 从文件创建ConfigMap
kubectl create configmap app-config --from-file=config.ini

# 从目录创建ConfigMap
kubectl create configmap app-config --from-file=./config/

# 从键值对创建ConfigMap
kubectl create configmap app-config --from-literal=DATABASE_URL=mysql://localhost --from-literal=CACHE_SIZE=1000

# 查看ConfigMap
kubectl get configmaps
kubectl describe configmap app-config

# 编辑ConfigMap
kubectl edit configmap app-config

# 删除ConfigMap
kubectl delete configmap app-config
```

### ConfigMap YAML示例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    server.port=8080
    logging.level=INFO
  database.conf: |
    host=localhost
    port=3306
---
# 在Pod中使用ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: CONFIG_PATH
      value: /etc/config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Secret

```bash
# 创建Secret（通用）
kubectl create secret generic my-secret --from-literal=password=mypassword

# 创建docker-registry Secret（用于私有镜像仓库）
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com

# 创建TLS Secret（用于HTTPS）
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/key.key

# 查看Secret
kubectl get secrets
kubectl describe secret my-secret

# 删除Secret
kubectl delete secret my-secret
```

### PersistentVolume和PersistentVolumeClaim

```bash
# 查看PersistentVolume
kubectl get pv

# 查看PersistentVolumeClaim
kubectl get pvc

# PersistentVolume YAML示例
cat > pv.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
EOF
kubectl apply -f pv.yaml

# PersistentVolumeClaim YAML示例
cat > pvc.yaml << 'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
kubectl apply -f pvc.yaml
```

---

## 实战场景

### 场景 1: 部署一个三副本的Java应用

**需求**: 部署Spring Boot应用，配置自动扩展和滚动更新

**YAML文件**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myregistry.azurecr.io/myapp:1.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: JAVA_OPTS
          value: "-Xms512m -Xmx1g"
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 2
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deploy
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**部署步骤**:
```bash
kubectl apply -f deployment.yaml
kubectl get pods -w
kubectl logs -f deployment/myapp-deploy
```

---

### 场景 2: 使用Helm包管理器部署应用

**需求**: 使用Helm简化应用部署和配置管理

**安装Helm**:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**创建Helm Chart**:
```bash
helm create myapp

# 修改values.yaml配置
# 部署
helm install myapp-release ./myapp

# 查看已安装的Release
helm list

# 升级
helm upgrade myapp-release ./myapp

# 回滚
helm rollback myapp-release 1

# 卸载
helm uninstall myapp-release
```

---

## 监控和调试

### 集群监控

```bash
# 查看集群资源使用情况
kubectl top nodes
kubectl top pods

# 按命名空间查看资源使用
kubectl top pods -n kube-system

# 查看Pod日志
kubectl logs <pod-name>

# 查看多个Pod的日志
kubectl logs <pod-name-1> <pod-name-2>

# 查看Pod事件
kubectl describe pod <pod-name>

# 查看Deployment事件
kubectl describe deployment <deployment-name>

# 查看事件列表
kubectl get events

# 按时间排序查看事件
kubectl get events --sort-by='.lastTimestamp'
```

### 问题排查

```bash
# 查看Pod状态为何未Running
kubectl describe pod <pod-name>

# 查看Pod为什么未被调度
kubectl get pods -o wide
kubectl describe pod <pending-pod>

# 进入Pod容器调试
kubectl exec -it <pod-name> -- /bin/bash

# 查看容器资源限制
kubectl describe pod <pod-name> | grep -A 5 "Limits"

# 查看Pod绑定的节点
kubectl get pods -o wide | grep <pod-name>

# 强制删除卡住的Pod
kubectl delete pod <pod-name> --grace-period=0 --force

# 查看kubelet日志（在节点上）
journalctl -u kubelet -f
```

---

## 快速命令参考

| 任务 | 命令 | 说明 |
|------|------|------|
| 查看集群信息 | `kubectl cluster-info` | 显示集群地址和版本 |
| 查看节点 | `kubectl get nodes` | 列出集群节点 |
| 查看Pods | `kubectl get pods -A` | 列出所有命名空间的Pod |
| 查看Deployment | `kubectl get deployments` | 列出Deployment |
| 创建Deployment | `kubectl create deploy name --image=image` | 快速创建Deployment |
| 查看日志 | `kubectl logs -f pod-name` | 实时查看Pod日志 |
| 进入容器 | `kubectl exec -it pod-name bash` | 进入Pod容器 |
| 查看资源 | `kubectl top nodes/pods` | 查看资源使用情况 |
| 应用配置 | `kubectl apply -f file.yaml` | 应用YAML配置 |
| 删除资源 | `kubectl delete pod pod-name` | 删除Pod |

---

## 更新日志

- **2025-11-24**: 初版完成，包含集群管理、资源管理、工作负载等完整内容