# CKA 练习题

- [CKA 练习题](#cka-练习题)
  - [1. RBAC](#1-rbac)
    - [Contex](#contex)
    - [Task](#task)
    - [参考](#参考)
  - [2. node-drain](#2-node-drain)
    - [Task](#task-1)
    - [参考](#参考-1)
  - [3. clusterupdate](#3-clusterupdate)
    - [Task](#task-2)
    - [参考](#参考-2)
  - [5. NetworkPolicy](#5-networkpolicy)
    - [Task](#task-3)
    - [参考](#参考-3)
  - [6. createService](#6-createservice)
    - [Task](#task-4)
    - [参考](#参考-4)
  - [7. ingress](#7-ingress)
  - [8. scaledeployment](#8-scaledeployment)
    - [Task](#task-5)
    - [参考](#参考-5)
  - [9. schedulePod](#9-schedulepod)
    - [Task](#task-6)
    - [参考](#参考-6)


## 1. RBAC

### Contex  

  为部署流水线创建一个新的 ClusterRole 并将其绑定到特定的 namespace 的特定 ServiceAccount

### Task

  创建一个名为 deployment-clusterrole 且仅允许创建以下资源类型的新 ClusterRole:

- Deployment
- StatefulSet
- DaemqnSet

在现有的namespace app-team1 中创建一个名为 cicd-token 的新 ServiceAccount

限于namespace app-team1 中，将新的ClusterRole deployment-clusterrole 绑定到新的 ServiceAccount cicd-token 。

### 参考

```bash
$ kubectl create sa cicd-token -n app-team1 

$ kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployments,statefulsets,daemonsets

$ kubectl -n app-team1 create rolebinding cicd-clusterrole --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token

$ kubectl describe clusterrole deployment-clusterrole

$ kubectl get clusterrole | grep deployment

$ kubectl get rolebinding -n app-team1

$ kubectl get rolebinding -n app-team1

$ kubectl describe rolebinding -n app-team1 
```

- __Q__
  1. 什么是集群的资源，什么是命名空间下的资源？
  2. rolebind、 clusterrole、serviceaccount等分别是什么

## 2. node-drain

### Task

  将名为 ek8s-node-1 的 node 设置为不可用, 并重新调度该 node 上所有运行的 pods。

### 参考

```bash
$ kubectl get nodes

$ kubectl cordon node1.k8s.training.com

$ kubectl drain node1.k8s.training.com --ignore-daemonsets --delete-emptydir-data --force
  ```

## 3. clusterupdate

### Task

  现有的 Kubernetes 集群的运行版本 1.22.1。仅将master 节点上的所有 Kubernetes 控制平面 和节点组件升级到版本 1.22.2。

  确保在升级之前 drain master 节点，并在升级后uncordon master 节点。

### 参考

```bash
$ kubeadm version

$ kubectl cordon master.k8s.training.com


```

## 5. NetworkPolicy

### Task

在现有的 namespace fubar 中创建一个名为allow-port-from-namespace的 新 NetworkPolicy.

确保新的 NetworkPolicy 允许 namespace corp-net中的 Pods 连接到 namespace fubar 中的 Pods 端口8080.
进一步确保indeed NerworkPolicy:

- __不允许__ 对没有在监听端口 8080 的 Pods 的访问
- __不允许__ 非来自namespace corp-net中的 Pods 的访问

### 参考

在官网中先 copy 模板 yaml 文件

```yaml
# example.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: fubar
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: corp-net
          podSelector:    # 此处不能加 - ，否则会不仅匹配命名空间，还会匹配pod，而不是匹配命名空间下的pod
            matchLabels: {}
      ports:
        - protocol: TCP
          port: 80   #题目中端口要求 8080 ，但是练习时不好测试，就用 80  
```

```bash
$ kubectl apply -f example.yaml

# 以下都为测试
$ kubectl describe networkpolicies allow-port-from-namespace -n fubar

$ kubectl create deployment demo-deploy --image=nginx --port=80 -n fubar

# 这个镜像可以curl ip:port，不过最好往自己的dockerhub里传个这样的镜像
$ kubectl run t2 -it --image=radial/busyboxplus -n corp-net 
  # curl ip:80  （能通）

$ kubectl run t2 -it --image=radial/busyboxplus -n training
  # curl ip:80  （不能通）
```

测试OK的话就可以把 yaml 文件里的改为 8080 ，然后 delete 了重新 apply

## 6. createService

### Task

请重新配置现有的 deployment front-end 并添加名为 http 的端口规范来公开现有容器 nginx 的端口 80/tcp

创建一个名为 front-end-svc 的新服务，以公开容器端口 http

配置此服务，以通过各个 pods 所在的节点上的 NodePort 来公开它们。

### 参考

（准备步骤）先创建练习用的 deploy front-end , yaml文件也可以在官网 copy
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-end
  labels:
    app: front-end
spec:
  replicas: 1
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
```
如果 front-end 中没有 http 的 80 端口，则 `kubectl edit deploy front-end`，在 containers 中添加
```yaml
ports:
  - containerPort: 80
    name: http
    protocol: TCP
```
然后正式开始

```yaml
# front-end-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: front-end-svc
spec:
  type: NodePort
  selector:
    app: front-end
  ports:
    - port: 80
      targetPort: http
      nodePort: 30007
```

或者

```bash
$ kubectl expose deployment front-end --port=80 --target-port=http --name=front-end-svc2 --type=NodePort
```

## 7. ingress

TODO 还没装 ingress , 再说吧

## 8. scaledeployment

### Task

将 deployment loadbalancer 扩展至 3 个 pods.

### 参考

```bash
$ kubectl scale --replicas=3 deployment/loadbalancer -n training
```

## 9. schedulePod

### Task

按如下要求调度一个 pod

- 名称 : nginx-kusc00401
- image : nginx
- Node selector : disk=spinning

### 参考

同样可以在官网 copy 

```yaml
# pod-schedule.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  nodeSelector:
    disk: spinning
```

```bash
$ kubectl apply -f pod-schedule.yaml

# 查看是否调度到了目标 Node
$ kubectl describe po nginx-kusc00401    # 看 Node
```
