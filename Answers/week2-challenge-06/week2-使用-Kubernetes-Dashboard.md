# 挑战：使用 Kubernetes Dashboard

点击页面右上方的 CREATE 按钮进行创建：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635817785401)

复制 `mysql-rc.yaml` 文件的内容到输入并创建，然后点击上传：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635817846410)

等待 mysql Pod 的创建：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635817904201)

mysql Pod 创建成功：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635817908479)

在 `/home/shiyanlou` 目录下新建 `mysql-svc.yaml` 文件并写入对应的代码，使用从文件创建方式进行创建：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635817995084)

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818058597)

参考 myweb-rc.yaml 文件的内容，使用从表单创建的方式进行创建，注意相应数据的填写，需要点开显示高级选项：

- 应用名称：myweb
- 容器镜像：registry-vpc.cn-hangzhou.aliyuncs.com/chenshi-kubernetes/tomcat-app:v1
- 标签：密钥为 app，值为 myweb
- 环境变量
	- 名称为 MYSQL_SERVICE_HOST，值为 mysql
	- 名称为 MYSQL_SERVICE_PORT，值为 3306

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818404322)

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818408905)

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818414888)

点击部署，需要等待 myweb Pod 创建，myweb Pod 创建成功：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818669542)

复制 myweb-svc.yaml 文件的内容到输入并创建，然后点击上传：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818754675)

查看 myweb Service 创建成功：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818910497)

最后通过浏览器能够成功访问 `172.18.0.2:30001/demo/index.jsp` 页面：

![图片描述](https://doc.shiyanlou.com/courses/uid1491336-20211102-1635818845890)
