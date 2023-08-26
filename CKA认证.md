# RBAC
## Contex  
  为部署流水线创建一个新的 ClusterRole 并将其绑定到特定的 namespace 的特定 ServiceAccount     
     
## Task
  创建一个名为 deployment-clusterrole 且仅允许创建以下资源类型的新 ClusterRole:    
  - Deployment
  - StatefulSet
  - DaemqnSet   
  在现有的namespace app-team1 中创建一个名为 cicd-token 的新 ServiceAccount   
     
  限于namespace app-team1 中，将新的ClusterRole deployment-clusterrole 绑定到新的 ServiceAccount cicd-token 。
    
## 参考
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


# node-drain
## Task
  将名为 ek8s-node-1 的 node 设置为不可用, 并重新调度该 node 上所有运行的 pods。   
     

## 参考
```bash
$ kubectl get nodes

$ kubectl cordon node1.k8s.training.com

$ kubectl drain node1.k8s.training.com --ignore-daemonsets --delete-emptydir-data --force
  ```
   
# clusterupdate
## Task
  现有的 Kubernetes 集群的运行版本 1.22.1。仅将master 节点上的所有 Kubernetes 控制平面 和节点组件升级到版本 1.22.2。   
     
  确保在升级之前 drain master 节点，并在升级后uncordon master 节点。   
     
## 参考
```bash
$ kubeadm version

$ kubectl cordon master.k8s.training.com

$ 
```

# NetworkPolicy
## Task
在现有的 namespace fubar 中创建一个名为allow-port-from-namespace的 新 NetworkPolicy. 

确保新的 NetworkPolicy 允许 namespace corp-net中的 Pods 连接到 namespace fubar 中的 Pods 端口8080.   
进一步确保indeed NerworkPolicy:   
- __不允许__ 对没有在监听端口 8080 的 Pods 的访问
- __不允许__ 非来自namespace corp-net中的 Pods 的访问

## 参考
在官网中先 copy 模板 yaml 文件
