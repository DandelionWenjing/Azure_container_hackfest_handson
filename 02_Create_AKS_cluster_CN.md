## 使用Azure Container Service 创建kubernetes 集群

1. 创建AKS集群  

打开cmd窗口，在本文档中我们较多使用Azure CLI工具来进行Azure服务的部署，命令为 “az”:  

* 设定对global Azure进行操作:  

（如果是mooncake Azure, --name选择AzureChinaCLoud） 

```
az cloud set --name AzureCloud
```

* 登陆Azure账号：  

```
az login
```  

[!az_login](image/az_login.png)  


将网址输入浏览器，出现如下界面：
 
将代码输入到代码框中
 
登陆自己的账号：
 
出现如下界面登陆成功：
 
cmd界面显示：
az login
To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code GG42PZ4JB to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "id": "f270f3f0-a81d-4673-b27a-b989fab83ca5",
    "isDefault": true,
    "name": "Visual Studio Enterprise",
    "state": "Enabled",
    "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "user": {
      "name": "wenzhao@microsoft.com",
      "type": "user"
    }
  }
]

•	请求订阅上的功能标记：
az provider register -n Microsoft.ContainerService
az provider register -n Microsoft.ContainerService
Registering is still on-going. You can monitor using 'az provider show -n Microsoft.ContainerService'

•	创建资源组：
创建资源组来放k8s集群，-n  集群名称，-l  集群创建地点；
az group create -n K8SCluster -l eastus
az group create -n K8SCluster -l eastus
{
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourceGroups/K8SCluster",
  "location": "eastus",
  "managedBy": null,
  "name": "K8SCluster",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}

•	在Azure上创建kubernetes集群:
-g 后接参数为资源组，-c后接参数为节点数量，--generate-ssh-keys表示会新建ssh key在本地：
默认情况下k8s的版本为1.7.7，也可以增加新的参数-k来确认版本
az aks create -g K8SCluster -n K8SDeployment -c 1 -k 1.7.7 --generate-ssh-keys
•	Azure portal后台查看：
建立此集群可能需要5分钟来建立，这取决于我们在哪里创建以及有什么操作。
这时我们可以进入Azure portal查看后台的创建操作：
进入网址portal.azure.com并登陆自己的Azure账号：
点击左侧资源组按钮，查看新建的资源组信息：

  
可以看到我们新建的资源组出现在portal中，同时有一个MC开头的资源组一起创建(如果还没有可以等待一下之后点击上方refresh进行刷新)：
 
点击K8SCluster资源组，我们可以看到我们部署的容器服务资源对象：
 
进一步进入部署的资源对象中：
 
这里我们可以看到资源组的信息，包括部署的地点，kubernetes的版本号，agent节点的个数，agent虚机的大小等等。同时还有API server address,假如我们需要将AKS资源与jenkins/VSTS等做devops的工具进行连接的话需要用到该地址与API server相连。

现在我们进入另一个MC开头的资源组，这个资源组是包含了我们创建服务得所有实体对象得资源组：
 
包括虚拟网络，路由表，网络安全组，以及agent节点等等；
这时我们已经在相应得节点上安装好了kubeDNS, heapster以及kubelets使得kubernetes集群已经可以直接工作了。
我们现在回到cmd工具中查看集群创建情况：
az aks create -g K8SCluster -n K8SDeployment -c 1 -k 1.7.7 --generate-ssh-keys
{| Finished ..
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourcegroups/K8SCluster/providers/Microsoft.ContainerService/managedClusters/K8SDeployment",
  "location": "eastus",
  "name": "K8SDeployment",
  "properties": {
    "accessProfiles": null,
    "agentPoolProfiles": [
      {
        "count": 1,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "nodepool1",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D1_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "K8SDeploym-K8SCluster-f270f3",
    "fqdn": "k8sdeploym-k8scluster-f270f3-c0f7ccd4.hcp.eastus.azmk8s.io",
    "kubernetesVersion": "1.7.7",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4f77xevGCOZPMRmtB626eymW5f6b1usJbp9Qd7O0wjZX9d/hCme98EuZ0wQ6d/BHlPNc51FysIX9mPaFPIQ1+szM0C8Uzr9HYxtkvs2EoJPKdoCtGRyLjTGKPdhdg+1w7h1P8MVMnZgzwhcvD0kxRLPxQ1LT9Scwq5kI0px0nvYTCojtdj6lIMuomHUs3mX/zpF6tHOcVlc3bNbXPojBqLeJiEDLzXPSRgsubfjmLgbvisc9qSDSGHpQMYWFnBm37SKL1xp4ClE0MWYOObGQyBEBnNOQ3i5aIRv5Xch/7+OIj63D2KVSANIJq3ayT4KOjtMy6wQEbQEoBWsIJTGbl wenjingzhao0124@outlook.com\n"
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "8e14b7bd-06a4-4e1b-9104-36f02ea9dbdb",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "K8SCluster",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
集群创建成功后可以看到如上的结果。

