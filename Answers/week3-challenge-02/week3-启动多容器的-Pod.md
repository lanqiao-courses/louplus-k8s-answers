# 挑战：启动多容器的 Pod

在 `/home/shiyanlou` 目录下新建 `nginx.yaml` 文件，并向其中写入如下内容：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
    - name: nginx-container
      image: registry.cn-hangzhou.aliyuncs.com/louplus-linux/nginx:1.9.1
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
    - name: time-container
      image: registry-vpc.cn-hangzhou.aliyuncs.com/chenshi-kubernetes/time:1.0
      volumeMounts:
        - name: html
          mountPath: /html
      command: ["/bin/bash", "time.sh"]
  volumes:
    - name: html
      emptyDir: {}
```

执行创建：

```bash
$ kubectl create -f nginx.yaml
pod/volume-pod created
```

检查两个容器中的文件内容是否一致：

```bash
kubectl exec -it volume-pod -c nginx-container -- cat /usr/share/nginx/html/index.html

kubectl exec -it volume-pod -c time-container -- cat /html/index.html
```

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211109-1636423530497)
