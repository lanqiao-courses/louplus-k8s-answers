# 挑战：使用 Kubectl 完成需求

- 使用 `create` 命令创建 nginx.yaml 文件中定义的资源对象。

```bash
$ kubectl create -f nginx/
deployment.extensions/nginx created
service/nginx created

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-68986b6d96-gdckn   1/1     Running   0          21s

$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           30s

$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        18d
nginx        NodePort    10.110.188.107   <none>        80:31001/TCP   2m47s
```

- 使用 `edit` 命令将 nginx service 的 nodePort 从 31001 修改为 31002，使得执行命令 `curl http://172.18.0.4:31002/` 可以成功获取到数据。

```bash
$ kubectl edit service nginx
service/nginx edited

$ curl http://172.18.0.4:31002/
<!DOCTYPE html>
...
<title>Welcome to nginx!</title>
...

# 检查之前的端口是否还可以使用
$ curl http://172.18.0.4:31001/
curl: (7) Failed to connect to 10.192.0.2 port 31001: 拒绝连接
```

- 使用 `patch` 命令将 nginx 的镜像更换为 `registry-vpc.cn-hangzhou.aliyuncs.com/chenshi-kubernetes/nginx:1.13-alpine`。

```bash
$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-68986b6d96-gdckn   1/1     Running   0          25m

$ kubectl patch pod nginx-68986b6d96-gdckn -p '{"spec":{"containers":[{"name":"nginx","image":"registry-vpc.cn-hangzhou.aliyuncs.com/chenshi-kubernetes/nginx:1.13-alpine"}]}}'
pod/nginx-68986b6d96-gdckn patched

# 确认更换以后 pod 中的 nginx 镜像版本
$ kubectl exec -it nginx-68986b6d96-gdckn -- /bin/sh
/ # nginx -v
nginx version: nginx/1.13.12
/ #
```

- 修改 nginx.yaml 文件，将 nginx service 的 nodePort 从 31002 修改为 31003，使用 `apply` 命令对资源对象进行更新。

```bash
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        18d
nginx        NodePort    10.110.188.107   <none>        80:31002/TCP   34m

$ vim nginx/nginx.yaml
$ grep 31003 nginx/nginx.yaml
    nodePort: 31003

$ kubectl apply -f nginx/
deployment.extensions/nginx configured
service/nginx configured

# 查看是否更改成功
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        18d
nginx        NodePort    10.110.188.107   <none>        80:31003/TCP   39m
```

- 使用 `scale` 命令将 nginx 资源对象进行横向扩展，从 1 个副本扩展为 3 个副本。

```bash
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-68986b6d96-gdckn   1/1     Running   1          43m   10.244.2.4   kubernetes-worker   <none>           <none>

$ kubectl scale --current-replicas=1 --replicas=3 deployment/nginx
deployment.extensions/nginx scaled

# 查看是否将 pod 副本扩展到了 3 个
$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           46m
$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-68986b6d96-bxpws   1/1     Running   0          24s   10.244.2.5   kubernetes-worker   <none>           <none>
nginx-68986b6d96-gdckn   1/1     Running   1          46m   10.244.2.4   kubernetes-worker   <none>           <none>
nginx-68986b6d96-r2ffx   1/1     Running   0          24s   10.244.3.3   kubernetes-workerer2   <none>           <none>
```

- 对节点 kubernetes-worker 执行维护操作之前，使用 `drain` 命令安全驱逐 kubernetes-worker 节点上面所有的 pod。

```bash
# 可以发现在 kubernetes-worker 节点上还运行着两个 nginx pod
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE          NOMINATED NODE   READINESS GATES
nginx-68986b6d96-bxpws   1/1     Running   0          110m   10.244.2.5   kubernetes-worker   <none>           <none>
nginx-68986b6d96-gdckn   1/1     Running   1          156m   10.244.2.4   kubernetes-worker   <none>           <none>
nginx-68986b6d96-r2ffx   1/1     Running   0          110m   10.244.3.3   kubernetes-workerer2   <none>           <none>

$ kubectl drain kubernetes-worker --ignore-daemonsets
node/kubernetes-worker cordoned # 设置 kubernetes-worker 不可使用
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-j9n7q
evicting pod "nginx-68986b6d96-bxpws" # 驱逐 pod
evicting pod "coredns-5c98db65d4-tb5fz"
evicting pod "nginx-68986b6d96-gdckn"
pod/nginx-68986b6d96-gdckn evicted
pod/nginx-68986b6d96-bxpws evicted
pod/coredns-5c98db65d4-tb5fz evicted
node/kubernetes-worker evicted

# 查看结果可以看到 kubernetes-worker 已经处于维护状态
$ kubectl get nodes
NAME          STATUS                     ROLES    AGE   VERSION
kube-master   Ready                      master   18d   v1.15.0
kubernetes-worker   Ready,SchedulingDisabled   <none>   18d   v1.15.0
kubernetes-workerer2   Ready                      <none>   18d   v1.15.0
# 在 kubernetes-workerer2 生成了两个新的 nginx pod，保证集群中依然有 3 个 nginx pod
$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE          NOMINATED NODE   READINESS GATES
nginx-68986b6d96-psp69   1/1     Running   0          55s    10.244.3.4   kubernetes-workerer2   <none>           <none>
nginx-68986b6d96-r2ffx   1/1     Running   0          113m   10.244.3.3   kubernetes-workerer2   <none>           <none>
nginx-68986b6d96-rg4l6   1/1     Running   0          55s    10.244.3.6   kubernetes-workerer2   <none>           <none>
```
