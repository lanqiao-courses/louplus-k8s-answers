# 挑战：使用 Helm 部署 Tomcat

添加仓库：

```bash
# 添加一个 Chart 仓库，这里设置阿里云 Chart 镜像仓库为标准仓库
helm repo add stable https://apphub.aliyuncs.com/
```

创建应用：

```bash
$ helm create tomcat
Creating tomcat
```

修改配置信息：

```bash
vim tomcat/templates/deployment.yaml
# 主要修改如下所示
      containers:
        - name: {{ .Chart.Name }}
          ...
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}" # 修改使用镜像标签
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8080 # 设置容器端口为 8080
              protocol: TCP
          #livenessProbe:   # 删除存活探针和就绪探针的检测
          #  httpGet:
          #    path: /
          #    port: http
          #readinessProbe:
          #  httpGet:
          #    path: /
          #    port: http

```

```bash
vim tomcat/values.yaml
...
image:
  repository: registry-vpc.cn-hangzhou.aliyuncs.com/chenshi-kubernetes/tomcat-app   # 修改镜像仓库地址
  tag: v1      # 设置镜像版本号
  pullPolicy: IfNotPresent
...
service:
  type: NodePort   # 将服务修改为 NodePort 类型
  port: 80
```

修改完成后，执行测试和安装：

```bash
# 测试
helm install --debug --dry-run test tomcat/
# 安装
helm install tomcat tomcat/
```

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638432005242)

访问测试：

```bash
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services tomcat)

export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")

echo http://$NODE_IP:$NODE_PORT
http://172.18.0.3:30152
```

复制上面获取到的链接地址，然后在浏览器打开即可访问：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638432386600)
