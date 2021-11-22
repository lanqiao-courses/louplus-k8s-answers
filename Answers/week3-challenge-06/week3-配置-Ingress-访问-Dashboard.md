# 挑战：配置 Ingress 访问 Dashboard

部署 nginx-ingress-controller 插件：

需要下载 `nginx-ingress-controller.yaml` 文件和 `ingress-service-nodeport.yaml` 文件，这里已经提供了完整的 YAML 文件，大家执行下载即可：

```bash
wget https://labfile.oss.aliyuncs.com/courses/1494/nginx-ingress-controller.yaml
wget https://labfile.oss.aliyuncs.com/courses/1457/ingress-service-nodeport.yaml
```

先创建 nginx-ingress-controller Pod：

```bash
kubectl create -f nginx-ingress-controller.yaml
```

然后为 nginx-ingress-controller Pod 创建 NodePort 类型的 Service：

```bash
kubectl create -f ingress-service-nodeport.yaml
```

查看 kubernetes-dashboard 的相关信息：

```bash
kubectl get all -n kubernetes-dashboard
```

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211122-1637572520469)

可以看到 kubernetes-dashboard service 的端口号为 443。

首先生成自签名证书：

```bash
# 指定域名为 dashboard.kube.com
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout kube-dashboard.key -out kube-dashboard.crt -subj "/CN=dashboard.kube.com"
Generating a 2048 bit RSA private key
..............................................+++
........................................+++
writing new private key to 'kube-dashboard.key'
-----
```

然后创建 Secret 资源对象：

```bash
$ kubectl create secret tls kube-dasboard-secret --key kube-dashboard.key --cert kube-dashboard.crt -n kube-system
secret/kube-dashboard-secret created
```

在 `/home/shiyanlou` 目录下新建 `kube-dashboard-ingress.yaml` 文件，并向其中写入如下内容：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-kube-dashboard
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS" # Ingress Controller 将客户端的请求转发给 dashboard 服务时使用 HTTPS 协议，如果这里不指定默认为 HTTP 协议，这样的话会报错
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  tls:
    - hosts:
        - dashboard.kube.com
      secretName: kube-dashboard-secret # HTTPS 证书 Secret
  rules:
    - host: dashboard.kube.com # 域名
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-dashboard # 服务名为 kubernetes-dashboard
                port:
                  number: 443 # 服务端口号为 443
```

执行创建：

```bash
$ kubectl create -f kube-dashboard-ingress.yaml
ingress.extensions/ingress-kube-dashboard created
$ kubectl get ingress -n kubernetes-dinashboard
NAME                     HOSTS                ADDRESS         PORTS     AGE
ingress-kube-dashboard   dashboard.kube.com   10.96.174.174   80, 443   34s
```

在本机添加域名解析，执行命令 `sudo vim /etc/hosts` 并在文件末尾添加如下内容：

```text
172.18.0.2  dashboard.kube.com
172.18.0.3  dashboard.kube.com
172.18.0.4  dashboard.kube.com
```

最后就可以在浏览器访问地址 `https://dashboard.kube.com` 啦。
