# 挑战：配置 Harbor 为 Helm 的存储仓库

安装 Helm：

```bash
# 添加官方 Chart 仓库
helm repo add stable https://charts.helm.sh/stable
# 添加 harbor 的 helm 仓库
helm repo add goharbor https://helm.goharbor.io
helm repo ls
helm repo update
```

使用 helm 安装 harbor：

```bash
# 安装 harbor，关闭持久存储以及 tls 认证，设置服务访问方式为 NodePort，并且指定外部访问地址为 http://172.18.0.3:30002
helm install harbor goharbor/harbor --set persistence.enabled=false --set expose.tls.enabled=false --set expose.type=nodePort --set externalURL=http://172.18.0.3:30002
```

等待 Pod 完全启动：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638434474023)

然后在浏览器访问 `172.18.0.3:30002` 地址，结果如下所示，默认账号为 `admin`，密码为 `Harbor12345`：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638434540093)

新建一个项目仓库，命名为 `chart_repo`，访问级别为公开：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638436453756)

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638436477489)

新建一个用户，用户名为 `shiyanlou`，邮箱为 `shiyanlou@simplecloud.com`，密码为 `Shiyanlou123`：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638436507599)

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211202-1638436709012)

添加仓库地址，并配置推送：

```bash
# 添加仓库地址
$ helm repo add test_repo http://172.18.0.3:30002/chartrepo/chart_repo
"test_repo" has been added to your repositories
$ helm repo ls
NAME        URL
stable      https://charts.helm.sh/stable
goharbor    https://helm.goharbor.io
test_repo   http://172.18.0.3:30002/chartrepo/chart_repo
```

安装 helm-push 插件，这里可能遇到 GitHub 访问超时严重的问题，可以逐个查找哪个 DNS 可以访问，然后配置到 `/etc/hosts` 中：

```bash
$ helm plugin install https://github.com/chartmuseum/helm-push
Downloading and installing helm-push v0.8.1 ...
https://github.com/chartmuseum/helm-push/releases/download/v0.8.1/helm-push_0.8.1_linux_amd64.tar.gz
Installed plugin: push

# 查看插件
$ helm plugin ls
NAME    VERSION DESCRIPTION
cm-push    0.10.1   Push chart package to ChartMuseum

# 获取一个 MySQL Chart 作为测试
$ helm fetch stable/mysql
$ ls
mysql-6.8.0.tgz
# 直接将打包好的 Chart 推送到远端仓库中，这里需要指定 Harbor 的用户名和密码
$ helm cm-push mysql-1.6.2.tgz test_repo -u admin -p Harbor12345
Pushing mysql-1.6.2.tgz to test_repo...
Done.
```

![images](https://doc.shiyanlou.com/courses/1527/600404/74682b181e7615f56d6b0f21b29c0bf2-0/wm)
