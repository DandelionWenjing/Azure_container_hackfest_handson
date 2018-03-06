## 管理kubernetes 集群  

#### 使用kubectl：  

1. 安装kubectl:  

查看本地kubectl是否已经安装，如果未安装，安装kubectl并添加路径:  

Windows下kubectl安装地址：  

https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/windows/amd64/kubectl.exe  

2. 配置Kubectl连接到k8s集群：  

```
az aks get-credentials -g K8SCluster -n K8SDeployment
```
默认会下载kube config文件在~/.kube/config中。  

当我们执行该命令时，会默认对相应的AKS集群进行操作。  

* 【如果在获取该命令的时候出现如下的错误，可能跟CLI的版本相关，更新一下CLI的版本再进行尝试】：  
```
No Kubernetes access profiles found. Cluster provisioning state is "Succeeded".
```
下载Azure CLI 2.0.23：  

https://azurecliprod.blob.core.windows.net/msi/azure-cli-2.0.23.msi  

安装新版本Azure CLI后再次执行如上命令：  
```
az aks get-credentials -g K8SCluster -n K8SDeployment
Merged "K8SDeployment" as current context in C:\Users\wenzhao\.kube\config
```
新的集群信息会merge进kube config file中。  

即使你使用minikube 也同样的被包含进配置信息文件中。  

3. 列出订阅下所有的AKS集群信息：  
```
az aks list -o table
```

列出结果如下：  
```
az aks list -o table
Name           Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
-------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------
K8SDeployment  eastus      K8SCluster       1.7.7                Succeeded            k8sdeploym-k8scluster-f270f3-c0f7ccd4.hcp.eastus.azmk8s.io
```

4. 获取集群节点情况：  
```
kubectl get nodes
```
结果如下：  
···
kubectl get nodes
NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-25044415-0   Ready     agent     1h        v1.7.7
```  

我们创建了一个agent node,所以这里可以看到agent node的信息，默认命名前缀包括这个agent node所在的agent pool;  

5. 集群的管理：  

获取kubernetes基本单位pods的所有信息：  
```
Kubectl get pods –all-namespaces
```
结果如下：  
```
kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE
kube-system   heapster-342135353-g62cc                2/2       Running   0          1h
kube-system   kube-dns-v20-1654923623-7d8cm           3/3       Running   0          1h
kube-system   kube-dns-v20-1654923623-9nc7n           3/3       Running   0          1h
kube-system   kube-proxy-t272w                        1/1       Running   0          1h
kube-system   kube-svc-redirect-cqwdl                 1/1       Running   0          1h
kube-system   kubernetes-dashboard-1672970692-k4n8g   1/1       Running   0          1h
kube-system   tunnelfront-851434796-0fhr4             1/1       Running   0          1h
```

我们可以看到在现有的kubernetes集群中能够已经存在的基本pods信息。  

这里包括有heapster做监控, kubeDNS, kube proxy等;  

6. 登陆kubernetes dashboard:  
```
az aks browse -g K8SCluster -n K8SDeployment
az aks browse -g K8SCluster -n K8SDeployment
Merged "K8SDeployment" as current context in C:\Users\wenzhao\AppData\Local\Temp\tmp47m9axp2
Proxy running on http://127.0.0.1:8001/
Press CTRL+C to close the tunnel...
Forwarding from 127.0.0.1:8001 -> 9090
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001
Handling connection for 8001

登陆网址127.0.0.1:8001即可看到kubernetes的dashboard（也可能会自动弹出）:
 
它会自动创建一个通道，自动打开这个dashboard,如果你对kubernetes的dashboard非常熟悉和适应的话，有很多操作可以直接在dashboard上面进行。
只要这个tunnel建立起来了，我们就可以直接在k8s的dashborad上面操作。
如果想要离开这个dashboard,可以CRTL+C来关闭这个通道。
•	对AKS集群做扩展：
az aks scale -g K8SCluster -n K8SDeployment -c 3

我们将集群中1个节点扩展到了3个节点，这一过程可能需要花费5分钟的时间。
但是在portal中，我们很快可以看到新建起来的虚机以及挂载在上面的磁盘：
 
az aks scale -g K8SCluster -n K8SDeployment -c 3
{\ Finished ..
  "agentPoolProfiles": [
    {
      "count": 3,
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
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourcegroups/K8SCluster/providers/Microsoft.ContainerService/managedClusters/K8SDeployment",
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
  "location": "eastus",
  "name": "K8SDeployment",
  "provisioningState": "Succeeded",
  "resourceGroup": "K8SCluster",
  "servicePrincipalProfile": {
    "clientId": "8e14b7bd-06a4-4e1b-9104-36f02ea9dbdb",
    "keyVaultSecretRef": null,
    "secret": null
  },
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}

•	获取kubernetes版本信息：
az aks get-versions -g K8SCluster -n K8SDeployment -o table
az aks get-versions -g K8SCluster -n K8SDeployment -o table
Name     ResourceGroup    MasterVersion    MasterUpgrades       NodePoolVersion    NodePoolUpgrades
-------  ---------------  ---------------  -------------------  -----------------  -------------------
default  K8SCluster       1.7.7            1.8.2, 1.7.9, 1.8.1  1.7.7              1.8.2, 1.7.9, 1.8.1
这个命令可以告诉我们现在的版本信息以及可以升级的版本信息。
•	做kubernetes升级：
az aks upgrade --name K8SDeployment --resource-group K8SCluster --kubernetes-version 1.8.2
这个步骤可能需要花费一定的时间，虽然提示unavailable但是现有的服务不会go down，只是不能部署新服务。
az aks upgrade --name K8SDeployment --resource-group K8SCluster --kubernetes-version 1.8.2
Kubernetes may be unavailable during cluster upgrades.
Are you sure you want to perform this operation? (y/n): y
{\ Finished ..
  "agentPoolProfiles": [
    {
      "count": 3,
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
  "id": "/subscriptions/f270f3f0-a81d-4673-b27a-b989fab83ca5/resourcegroups/K8SCluster/providers/Microsoft.ContainerService/managedClusters/K8SDeployment",
  "kubernetesVersion": "1.8.2",
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
  "location": "eastus",
  "name": "K8SDeployment",
  "provisioningState": "Succeeded",
  "resourceGroup": "K8SCluster",
  "servicePrincipalProfile": {
    "clientId": "8e14b7bd-06a4-4e1b-9104-36f02ea9dbdb",
    "keyVaultSecretRef": null,
    "secret": null
  },
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}

•	确认是否升级成功：
az aks show --name K8SDeployment --resource-group K8SCluster --output table
az aks show --name K8SDeployment --resource-group K8SCluster --output table
Name           Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
-------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------
K8SDeployment  eastus      K8SCluster       1.8.2                Succeeded            k8sdeploym-k8scluster-f270f3-c0f7ccd4.hcp.eastus.azmk8s.io

