管理 Kubernetes 集群上的软件包,按顺序创建kubernetes中的实例与各种资源





安装

```shell
➜  brew install helm
```



概念

- Chart：包含工具或服务所需要的定义（类似于image）
- Repository：存放和共享Chart（类似于hub.docker）
- Release：运行的实例（类似于container）



查找chart

```shell
# 从公共制品库中查找
➜  config helm search hub redis
URL                                               	CHART VERSION	APP VERSION     	DESCRIPTION
https://artifacthub.io/packages/helm/pascaliske...	0.0.2        	6.2.6           	A Helm chart for Redis
https://artifacthub.io/packages/helm/drycc-cana...	1.0.0        	                	A Redis database for use inside a Kubernetes cl...
https://artifacthub.io/packages/helm/truecharts...	1.0.54       	6.2.6           	Open source, advanced key-value store.

# 从本地添加的制品库中查找
➜  helm repo add bitnami https://charts.bitnami.com/bitnami
➜  helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkinsrepo" chart repository
Update Complete. ⎈Happy Helming!⎈
➜  config helm search repo redis
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/redis        	15.7.5       	6.2.6      	Open source, advanced key-value store. It is of...
bitnami/redis-cluster	7.1.2        	6.2.6      	Open source, advanced key-value store. It is of...
```





安装charts

```shell
# 默认配置安装
➜  helm install my-redis bitnami/redis
# 查看安装状态
➜  helm status my-redis

# 查看应用可配置项（yaml格式默认配置信息）
➜  shells helm show values bitnami/redis
# 指定配置安装（auth.password是yaml中路径）
➜  helm install my-redis --set auth.password=mypwd bitnami/redis
# 指定yaml配置安装（--values或-f）
➜  helm install my-redis --values redis.yaml bitnami/redis

```

