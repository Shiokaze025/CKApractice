# 1. kube-bench 修复不安全项
1. kube-apiserver.yaml修改
2. 在systemctl status kubelet查看配置文件在哪里
3. kubelet配置文件中，Webhook首字母需要大写  
4. etcd.yaml修改

查日志 `journalctl -xn` 或 `/var/log/pods/...` 

可选： `kube-bench --config-dir /目录 master`


# 2. Pod 指定 ServiceAccount
**任务——配置Pods和容器——为Pod配置服务账号**
1. edit sa 添加 `automountServiceAccountToken: false`
2. 修改 pod 的服务账号，绑定


# 3. 默认网络策略
**概念——服务、负载均衡和联网——网络配置——默认拒绝所有入站/出站流量**

# 4. RBAC - Rolebinding
1. `kubectl explain role`来补充`role.rules`下的内容，其中`apiGroups`是必填的
2. `kubectl create -n db role role-2 --verb=delete --resource=namespace`命令中`--verb`和`--resource`不需要后面加`s`，但是`namespaces`后面需要，且`=`后无需`[""]`引起来，直接使用`，`即可

# 5. ⽇志审计log audit
**任务——监控、日志和调试——集群故障排查——审计**
1. 审计配置直接copy
2. `kube-apiserver.yaml`修改，应该只有4项
3. 重启kubelet，`systemctl daemon-reload` ，`systemctl restart kubelet`

# 6. 创建Secret
**任务——管理Secrets**
**概念——配置——Secret**
1. 获取secret的方式：  
   ```shell
   $ kubectl get -n istio-system get secret db1-test -o jsonpath='{.data}'
   $ echo -n 'aGVsbG8=' | base64 --decode > /cks/sec/pass.txt 
   ```
2. 创建secret
   ```shell
   $ kubectl create -n istio-system secret generic --help
   $ kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret
   ```
3. 创建Pod，直接copy文档  

# 7. Dockerfile检测
1. 修改Dockerfile   
   - 系统镜像的版本不能是`latest`
   - 后面`CMD`有一段脚本执行，可能需要非特权用户，所以把执行用户改为`nobody`
2. 修改yaml文件   
   - template的标签和selector的标签对不上，需要修改下
   ```yaml
   {
      ...
      selector:
        matchLables:
          app: couchdb    # 需要一致
          version: stable
      template: 
        metadata:
          lables:
            app: couchdb     # 需要一致
            version: stable
      ...
   }
   ```
   - 修改安全上下文内的字段securityContext
   ```yaml
   {
      ...
      'privileged': False
      'readOnlyRootFilesystem': True
      'runAsUser': 65535
      ...
   }
   ```

# 8. 沙箱运行容器gVisor
**概念——容器——容器运行时**   
直接在文档copy

# 9. 网络策略NetworkPolicy
**概念——服务、负载均衡和联网——网络策略——按标签选择多个名字空间**  
和参考文档不一致，待确定

# 10. Trivy扫描镜像安全漏洞
1. 先把namespace下的pod扫描出来，然后获取镜像
```shell
$ kubectl describe -n kamino po | grep -iE 'name:|image:'
```

2. 扫描镜像，并删除有高风险的pod
```shell
$ trivy image --help

# 获得帮助的命令
$ trivy image --severity HIGH,CRITICAL alpine:3.15

$ for i in (amazonlinux:1,amazonlinux:2,nginx:1.19,vicuu/nginx:host); do trivy image -s HIGH,CRITICAL $i > &i.txt; done
```

# 11. AppArmor
**教程——安全——使用AppArmor限制容器对资源的访问**  
1. 查看apparmor_status
2. 



# 16. ImagePolicy Webhook 容器镜像扫描
**参考——API访问控制——准入控制器——ImagePolicyWebhook**

kube-apiserver.yaml中的两个属性配置还没找到，一个是定义准入控制器，一个是添加配置文件的目录。  

其他内容都可以在文档中找到