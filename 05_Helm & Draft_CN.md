## 在部署过程中使用helm和draft工具

Helm 和 Draft的使用：
•	Helm和draft的安装:
Helm是一个包管理器；可以安装一些打包好的系统或者应用程序包。
Helm有两个部分，一个是client端叫helm,另一个是server端叫Tiller.
我们首先需要安装helm和tiller:
Helm github 安装文档：
https://github.com/kubernetes/helm/blob/master/docs/install.md
Helm release 地址：
https://github.com/kubernetes/helm/releases
安装完成之后在命令行中尝试helm是否正确安装。

helm init
Creating C:\Users\wenzhao\.helm
Creating C:\Users\wenzhao\.helm\repository
Creating C:\Users\wenzhao\.helm\repository\cache
Creating C:\Users\wenzhao\.helm\repository\local
Creating C:\Users\wenzhao\.helm\plugins
Creating C:\Users\wenzhao\.helm\starters
Creating C:\Users\wenzhao\.helm\cache\archive
Creating C:\Users\wenzhao\.helm\repository\repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at C:\Users\wenzhao\.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
Happy Helming!

这时我们查看一下在kubenetes集群里面的系统pod,可以发现有Helm server:Tiller
kubectl -n kube-system get pod
NAME                                    READY     STATUS    RESTARTS   AGE
heapster-342135353-g62cc                2/2       Running   0          5h
kube-dns-v20-1654923623-7d8cm           3/3       Running   0          5h
kube-dns-v20-1654923623-9nc7n           3/3       Running   0          5h
kube-proxy-h700j                        1/1       Running   0          3h
kube-proxy-pptt2                        1/1       Running   0          3h
kube-proxy-t272w                        1/1       Running   0          5h
kube-svc-redirect-8gd16                 1/1       Running   0          3h
kube-svc-redirect-cqwdl                 1/1       Running   0          5h
kube-svc-redirect-mdm21                 1/1       Running   0          3h
kubernetes-dashboard-1672970692-k4n8g   1/1       Running   0          5h
tiller-deploy-352283156-zfqpl           1/1       Running   0          1m
tunnelfront-851434796-0fhr4             1/1       Running   0          5h

Draft是帮助我们更方便的构建程序的工具：
Draft能直接帮助我们生成helm charts.然后部署到Kubernetes集群上，当你在进行code的时候它即能够帮助你快速进行测试，更改完代码直接draft up draft create即可。我们也可以把draft和repository绑定在一起，它可以创建容器，推进repository,再部署到k8s上面。这就帮助我们构建了devops的pipeline.

Draft也分两个部分，client端叫draft,server端叫draft-d.
我们需要安装draft和draft-D：
Draft release地址:
https://github.com/Azure/draft/releases

•	使用helm和draft构建程序：
这里我们可以使用helm来部署ingress-controller 到 AKS集群中;

下载下列地址的nginx-ingress controller的文件：
https://github.com/kubernetes/charts/tree/master/stable/nginx-ingress

安装稳定版本ingress-controller:
helm install stable/nginx-ingress
helm install stable/nginx-ingress
NAME:   jazzy-macaw
LAST DEPLOYED: Mon Jan  8 21:43:44 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                                  DATA  AGE
jazzy-macaw-nginx-ingress-controller  1     3s

==> v1/Service
NAME                                       TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)                     AGE
jazzy-macaw-nginx-ingress-controller       LoadBalancer  10.0.216.180  <pending>    80:32225/TCP,443:31652/TCP  3s
jazzy-macaw-nginx-ingress-default-backend  ClusterIP     10.0.95.6     <none>       80/TCP                      3s

==> v1beta1/Deployment
NAME                                       DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
jazzy-macaw-nginx-ingress-controller       1        1        1           0          3s
jazzy-macaw-nginx-ingress-default-backend  1        1        1           0          3s

==> v1/Pod(related)
NAME                                                        READY  STATUS             RESTARTS  AGE
jazzy-macaw-nginx-ingress-controller-1444582388-fjl7f       0/1    ContainerCreating  0         3s
jazzy-macaw-nginx-ingress-default-backend-3715337617-njc32  0/1    ContainerCreating  0         3s

NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w jazzy-macaw-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

查看安装情况：
helm list
helm list
NAME            REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
jazzy-macaw     1               Mon Jan  8 21:43:44 2018        DEPLOYED        nginx-ingress-0.8.22    default

不论我们部署什么服务到kubernetes中，只要这个服务有Load balancer 的type:
那么这个服务就会自动调用Azure API创建一个load balancer，暴露一个公共IP给这个服务。这个服务会自动创建一个负载均衡规则。
我们现在再回到portal上面去看，就会看到一个名叫kubernetes的load balancer;
 

 


•	下载应用程序并构建：
现在我们需要用到的应用程序包含一些小的 API，我们需要写内容到运行Mongo DB API的cosmos DB 中。基本上是Mongo API for cosmos DB.现在我们有一个很简单的server.js文件，包含express, MongoDB 端, parser, 以及一个config文件，我们希望利用这个程序简单连接到数据库。
数据库使用环境变量：connection string;因为我们需要进入容器或者进入pod,环境变量是为了这个connection string的.我们将会使用Kubernetes secret来做这件事情，这样的话我们就不需要暴露任何的关于数据库的secret到我的代码中来，这使得我的代码有很好的可移植性。
程序中包含一些路由：有一些基本的操作，GET，DELETE,POST,PUT等。
程序里还有一个dockerfile,利用容器，我们可以本地部署，也可以云端部署，各处部署，这就是容器的好处。
运行git clone命令下载应用程序：
git clone https://github.com/DandelionWenjing/gbbapi.git
删掉chart,.draftignore,draft.toml文件。
•	配置draft:
首先我们需要配置一下draft:
初始化draft:
Draft init:
在这个过程中我们需要输入 acr login server, acr name 以及acr password;
In order to configure Draft, we need a bit more information...

1. Enter your Docker registry URL (e.g. docker.io/myuser, quay.io/myuser, myregistry.azurecr.io): registry0124.azurecr.io
2. Enter your username: Registry0124
3. Enter your password:
Draft has been installed into your Kubernetes Cluster.
Happy Sailing!

•	Draft创建应用程序charts：
下载如下应用程序：
https://github.com/DandelionWenjing/draft_demo.git

draft create
draft create
--> Draft detected the primary language as JavaScript with 93.620072% certainty.
--> Ready to sail
会出现ready to sail的结果，表示create成功; 可以看到draft create可以检索我们的编程语言，自动检测到我们是javascript语言。
同时在文件夹下会新出现三个文件：
 
Draft.toml: 这个文件详细描述了我们想要运行的文件，包括名称和所属的命名空间。
它创建了一个随机的名称，这里我们可以更改名称。
[environments]
  [environments.development]
    name = "hoping-molly"
    namespace = "default"
    wait = false
    watch = false
    watch_delay = 2
如果我们用该文件去做部署，那么服务的名称就是hoping-molly.
下面的watch是一个非常cool的功能，它可以等待我们的文件被更改，之后做自动重新构建和重新push的工作，相当于一个自动的draft up。这里我们可以先设置为false.
Chart文件：表示这是一个要部署到kubernetes中的helm charts.
其中的values.yaml描述了一些变量。
chart.yaml是一些描述文件。
Templates包含一些模板文件，比如描述了我们如何部署一个deployment资源对象等。
•	Draft构建应用程序：
如果我们想要进一步自定义我们的应用，可以更改我们的应用程序，更改后的代码下载如下：
下载自定义好的文件做应用程序build:
https://github.com/DandelionWenjing/gbbapi.git
由于该应用程序需要cosmos DB服务，我们在Azure portal中创建cosmos DB服务：
•	添加cosmos DB服务：
搜索cosmos DB:
 
点击create;

 
点击create;
创建完成cosmos DB后，将connection string复制下来：
 
粘贴到如下网站转成Base64码：
https://www.base64encode.org/
 
记录下该码供我们之后使用；
•	使用draft up构建：
draft up
我们想要基于我现在的dockerfile build我的应用程序。
把我们刚才记录下来的connection string填写到代码中：
 
 

构建完成了之后我们的应用程序镜像会upload到draft之前绑定的registry上面，这里我们在前面绑定的是Azure container registry.
它自动创建了一个secret.
首先它会部署我们的secret文件，接着会部署工作负载，即我们的deployment和service到集群上。
大概需要1分多钟使用dockerfile build这个镜像,需要几十秒钟将镜像推进registry.
draft up
[94;40;4mDraft Up Started[0m: '[36;40mgbb-api[0m'
[36;40mgbb-api[0m: [92;40;1mBuilding Docker Image[0m: SUCCESS ⚓  (37.0373s)
[36;40mgbb-api[0m: [92;40;1mPushing Docker Image[0m: SUCCESS ⚓  (29.0741s)
[36;40mgbb-api[0m: [92;40;1mReleasing Application[0m: SUCCESS ⚓  (0.4811s)
[36;40mgbb-api[0m: [94;40;4mBuild ID[0m: [93;40;1m01C3D477BPVY2YH8F8D8C0SHGC[0m
可能会有乱码，但是可以看到一步步的过程和时间。

再次运行kubectl get pods:
kubectl get pods
NAME                                                         READY     STATUS    RESTARTS   AGE
azure-vote-back-4149398501-xfjn6                             1/1       Running   0          7h
azure-vote-front-1874756303-778wg                            1/1       Running   0          2h
azure-vote-front-1874756303-j6wsj                            1/1       Running   1          19h
azure-vote-front-1874756303-k8qpd                            1/1       Running   0          2h
azure-vote-front-1874756303-mswvz                            1/1       Running   0          2h
azure-vote-front-1874756303-sdv6f                            1/1       Running   0          2h
gbb-api-javascript-c4d9797b4-2mnmw                           1/1       Running   0          1m
gbb-api-javascript-c4d9797b4-9nj28                           1/1       Running   0          1m
jazzy-macaw-nginx-ingress-controller-1444582388-gj5qm        1/1       Running   1          19h
jazzy-macaw-nginx-ingress-default-backend-3715337617-xxzdp   1/1       Running   0          7h
可以看到新的pods已经部署上去了。
Kubectl get secret:
kubectl get secret
NAME                       TYPE                                  DATA      AGE
default-token-pzvk1        kubernetes.io/service-account-token   3         1d
draftd-pullsecret          kubernetes.io/dockercfg               1         2m
my-cosmosdb-mongo-secret   Opaque                                1         2m
我们现在创建了一个带有endpoint的服务，这个endpoint是有负载均衡的。
kubectl get svc
NAME                                        TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
azure-vote-back                             ClusterIP      10.0.86.16     <none>           6379/TCP                     21h
azure-vote-front                            LoadBalancer   10.0.254.34    40.121.195.128   80:31754/TCP                 21h
gbb-api-javascript                          LoadBalancer   10.0.35.70     52.170.232.30    8000:31638/TCP               14m
jazzy-macaw-nginx-ingress-controller        LoadBalancer   10.0.216.180   13.72.104.168    80:32225/TCP,443:31652/TCP   20h
jazzy-macaw-nginx-ingress-default-backend   ClusterIP      10.0.95.6      <none>           80/TCP                       20h
kubernetes                                  ClusterIP      10.0.0.1       <none>           443/TCP                      1d

可以使用draft connect来建立一个代理：
draft connect
Connecting to your app...SUCCESS...Connect to your app on localhost:60173
Starting log streaming...
We are live on port 8000
这时候我们先判断这个IP是否健康：
Localhost:60173/health
返回值为OK。
之后我们测试真正的public IP:
52.170.232.30:8000/health
返回OK.
现在我们来测试一下这个API：
下载postman:
https://www.getpostman.com/
发起post请求：
 
发起get 请求：
 
之后我们进入到cosmos DB中可以看到我们的id存储信息：
 

