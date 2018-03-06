## 部署容器化应用到Kubernetes集群中

### 从docker hub拉取镜像进行部署

示例应用程序是一个投票程序，包含前端和后端。  

kubernetes会根据yaml文件中的配置进行应用程序的部署，所以yaml文件非常重要。  

我们下载现有的yaml文件，包含资源对象deployment和service,做k8s的部署。  

#### 下载文件azure-vote.yml：  

从github下载包含了前端和后端的service和deployment:  
```
git clone https://github.com/DandelionWenjing/aks.git
```

#### 根据文件创建部署和服务： 

```
cd aks
kubectl create -f azure-vote.yml
```
得到结果如下：

kubectl create -f azure-vote.yml
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created

•	测试应用程序在云端运行情况：
kubectl get service azure-vote-front --watch
kubectl get service azure-vote-front --watch
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.254.34   <pending>     80:31754/TCP   2m
azure-vote-front   LoadBalancer   10.0.254.34   40.121.195.128   80:31754/TCP   3m
azure-vote-front的服务会暴露对外的IP地址，通过该IP地址我们可以使用该应用程序的服务。
浏览器中输入EXTERNAL-IP可以得到应用程序的运行界面：
 
监视完成之后可以ctrl+c停止监视。

### 打包应用程序到镜像仓库并进行部署

接下来我们对自定义应用程序进行打包，创建应用程序镜像，将镜像推到镜像仓库中，再使用aks从Azure的镜像仓库中把镜像部署到kubernetes集群中。  

#### 将应用程序下载到本地：
```
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
cd azure-voting-app-redis
```

#### 用compose文件创建容器镜像：

查看docker-compose.yml:
version: '3'
services:
  azure-vote-back:
    image: redis
    container_name: azure-vote-back
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "8080:80"

最后的ports是绑定本地8080端口到容器的80端口。如果本地的8080端口被占，可以更换成其他的端口。
docker-compose up -d
 
使用http://localhost:8080测试，如果8080端口被占了的话就换端口号。

部署使用Azure Container Registry:
•	创建资源组:
az group create -n Registry -l eastus
az group create -n Registry -l eastus
{
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourceGroups/Registry",
  "location": "eastus",
  "managedBy": null,
  "name": "Registry",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}

•	创建Azure容器注册表：
az acr create -g Registry -n Registry0124 --sku Basic
az acr create -g Registry -n Registry0124 --sku Basic
 - Finished ..
Create a new service principal and assign access:
  az ad sp create-for-rbac --scopes /subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourceGroups/Registry/providers/Microsoft.ContainerRegistry/registries/Registry0124 --role Owner --password <password>

Use an existing service principal and assign access:
  az role assignment create --scope /subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourceGroups/Registry/providers/Microsoft.ContainerRegistry/registries/Registry0124 --role Owner --assignee <app-id>
{
  "adminUserEnabled": false,
  "creationDate": "2018-01-09T01:49:31.127866+00:00",
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourceGroups/Registry/providers/Microsoft.ContainerRegistry/registries/Registry0124",
  "location": "eastus",
  "loginServer": "registry0124.azurecr.io",
  "name": "Registry0124",
  "provisioningState": "Succeeded",
  "resourceGroup": "Registry",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}

•	登陆容器注册表：
az acr login --name Registry0124
az acr login --name Registry0124
Login Succeeded

注意登陆的时候要保证docker正常运行；
并且用非normal的cmd可能会出现incorrect function.
•	显示loginserver和密码：
az acr show --name Registry0124 --query loginServer
az acr show --name Registry0124 --query loginServer
"registry0124.azurecr.io"

az acr credential show --name Registry0124 --query passwords[0].value
az acr credential show --name Registry0124 --query passwords[0].value
"VrtsqBGuZ8HV+UcYLH=zWfEWUTq8I2tc"

•	将镜像打标签推送到镜像仓库：
docker tag azure-vote-front registry0124.azurecr.io/azure-vote-front:redis-v1
docker push registry0124.azurecr.io/azure-vote-front:redis-v1
docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front                           latest              8a011828251c        33 minutes ago      935MB
registry0124.azurecr.io/azure-vote-front   redis-v1            8a011828251c        33 minutes ago      935MB
tiangolo/uwsgi-nginx-flask                 python3.6           cb32ec9f1c26        3 weeks ago         935MB
redis                                      latest              1e70071f4af4        4 weeks ago         107MB

•	返回推送到容器注册表的结果：
az acr repository list --name Registry0124 --output table
az acr repository list --name Registry0124 --output table
Result
----------------
azure-vote-front

运行应用程序：
下面再将应用程序部署到容器集群上，与之前从docker hub拉镜像不同的是，这次我们是从registry里面拉镜像，所以把镜像拉取地址换成registry的loginserver;
进入voting文件夹下，部署相应的service和deployment：
kubectl create -f azure-vote-all-in-one-redis.yml
 
后续步骤有关如何查看service，找IP，与之前类似；

