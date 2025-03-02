# BMC HelixOM ITOM & ITSM OnPrem Installation Step by Step 2 - ITOM安装

## 1 安装环境准备
### 1.1 Helix Deployment Manager下载
Helix ITOM安装过程由helix deployment manager完成。

* 登录[EPD](https://webepd.bmc.com/edownloads/ddl/cv/LP/442432/537020?fltk_=VTH1iwPCxfU%3D)，下载最新版本的helix-on-prem-deployment-manager-<release_version>.sh文件，例如helix-on-prem-deployment-manager-25.1.00-45.sh

![a3dfaebecc8a821b8ad7196eb517b21a.png](en-resource://database/683:1)

* 上传helix-on-prem-deployment-manager-25.1.00-45.sh到helix-svc服务器

* 增加shell文件的可执行权限

```
chmod a+x helix-on-prem-deployment-manager-25.1.00-45.sh
```

* 执行解压缩，创建目录helix-on-prem-deployment-manager
```
./helix-on-prem-deployment-manager-25.1.00-45.sh
```

* 修改目录名，方便版本区分
```
mv helix-on-prem-deployment-manager helix-on-prem-deployment-manager-25.1
```

### 1.2 配置文件设置
#### 1.2.1 infra.config

* 编辑./config/infra.config文件，修改如下表所示的参数值：

| 行号 | 参数 | 参数值 | 说明 |
| --- | --- | --- | --- |
| 9 | IMAGE_REGISTRY_HOST | helix-harbor.bmc.local |  |
| 10 | IMAGE_REGISTRY_USERNAME | admin | 密码在secrets文件中设置|
| 20 | NAMESPACE | helixade | itom安装的命名空间 |
| 21 | LB_HOST | lb.bmc.local | |
| 22 | LB_PORT | 443 | |
| 23 | TMS_LB_HOST | tms.bmc.local | |
| 24 | DOMAIN | bmc.local | |
| 26 | MINIO_LB_HOST | minio.bmc.local |  |
| 27 | MINIO_API_LB_HOST | minio-api.bmc.local |  |
| 28 | MINIO_API_LB_HOST | minio-api.bmc.local |  |
| 30 | TENANT_ENVIRONMENT | poc |  |
| 46 | TENANT_NAME | adelab |  |
| 47 | TENANT_EMAIL | adelab@bmc.local |  |
| 48 | TENANT_FIRST_NAME | adelab |  |
| 49 | TENANT_LAST_NAME | helix |  |
| 51 | TENANT_TYPE | private | |
| 53 | TENANT_COUNTRY | "China" |  |
| 68 | SMTP_HOST | helix-svc.bmc.local |  |
| 69 | SMTP_PORT | 25 |  |
| 70 | SMTP_USERNAME | dummy |  |
| 71 | SMTP_FROM_EMAIL | helix@bmc.local |  |
| 73 | SMTP_TLS | false |  |
| 74 | SMTP_AUTH_DASHBOARD | true |  |
| 75 | SMTP_AUTH  | PLAIN |  |
| 76 | OPS_GROUP_EMAIL | ops@bmc.local |  |
| 77 | APPROVAL_GROUP_EMAIL | approval@bmc.local |  |
| 91 | PG_STORAGE_CLASS | nfs-storage |  |
| 92 | VMSTORAGE_STORAGE_CLASS | nfs-storage |  |
| 93 | VMAGGSTORAGE_STORAGE_CLASS | nfs-storage |  |
| 94 | ES_MASTER_STORAGE_CLASS | nfs-storage |  |
| 95 | ES_DATA_STORAGE_CLASS | nfs-storage |  |
| 96 | MINIO_STORAGE_CLASS | nfs-storage |  |
| 97 | EFS_STORAGE_CLASS | nfs-storage |  |
| 98 | REDIS_HA_GLOBAL_STORAGECLASS | nfs-storage |  |
| 99 | KAFKA_STORAGECLASS | nfs-storage |  |  
| 100 | REDIS_CLUSTER_STORAGE_CLASS | nfs-storage |  |
| 101 | AIOPS_STORAGE_CLASS | nfs-storage |  |
| 105 | OPT_STORAGE_CLASS | nfs-storage |  |
| 111 | CUSTOM_CA_SIGNED_CERT_IN_USE | true |  |
| 126 | SMART_SYSTEM_USERNAME | system | Helix Discovery密码在secrets文件设置 |
| 130 | INGRESS_CLASS | nginx |  |
| 135 | INGRESS_TLS_SECRET_NAME | my-tls-secret |  |
| 140 | HELM_BIN | /usr/local/bin/helm |  |
| 141 | KUBECTL_BIN | /usr/local/bin/kubectl |  |
| 189 | LOGIN_ID | hannah_admin | Helix Dashboard登录用户 |


#### 1.2.2 deployment.config
编辑./config/deployment.config配置文件，修改如下参数：

| 行号 | 参数名 | 参数值 | 说明 |
| --- | --- | --- | --- |
| 7 | DEPLOYMENT_SIZE | compact |  |


#### 1.2.3 secrets.txt

* 编辑./common/certs/secrets.txt文件，修改如下内容：

| 行号 | 参数名 | 参数值 | 说明 |
| --- | --- | --- | --- |
| 2 | IMAGE_REGISTRY_PASSWORD | bmcAdm1n | Harbor的控制台登录密码 |
| 3 | SMTP_PASSWORD | dummy | MailHog邮箱不需要密码 |
| 4 | SMART_SYSTEM_PASSWORD | bmcAdm1n |  |
| 9 | ES_JKS_PASSWORD | bmcAdm1n | |

* secrets.txt文件在首次运行Helix安装程序时会被删除，建议一定要做好备份
```
cp secrets.txt secrets.txt.bak
```
#### 1.2.4 custom_cacert.pem
```
cp /root/opensslfull_chain.crt /root/helix-on-prem-deployment-manager-25.1/commons/../commons/certs/custom_cacert.pem
```
### 1.4 NFS块存储

#### 1.4.1 创建NFS服务器
* 在helix-svc安装NFS服务器软件
```
dnf install nfs-utils -y
```

* 创建NFS存储目录
```
mkdir -p /opt/datastore/helixade
chown -R nobody:nobody /opt/datastore/helixade
chmod -R 777 /opt/datastore/helixade
```

* 导出目录
```
echo "/opt/datastore/helixade  192.168.1.0/24(rw,sync,root_squash,no_subtree_check,no_wdelay)" > /etc/exports
exportfs -rv
```

* 为NFS开放防火墙
```
firewall-cmd --zone=internal --add-service mountd --permanent
firewall-cmd --zone=internal --add-service rpc-bind --permanent
firewall-cmd --zone=internal --add-service nfs --permanent
firewall-cmd --reload
```

* 启动NFS服务
```
systemctl enable nfs-server rpcbind
systemctl start nfs-server rpcbind nfs-mountd
```

* 验证NFS目录
```
showmount -e
```

#### 1.4.2 创建NFS StorageClass

* NFS Storage Class的创建，可以参考文档[nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

```
#Create namespace for storageclass
kubectl create namespace infra
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.1.1 --set nfs.path=/opt/datastore/helixade --set storageClass.name=nfs-storage -n infra
```

* 验证是否创建成功
```
kubectl -n infra get pod
kubectl get sc
```
### 1.5 安装HAProxy作为Load Balancer

* 安装HAProxy
```
dnf install haproxy -y
```

* 拷贝配置文件
```
\cp ~/helix-metal-install/haproxy.cfg /etc/haproxy/haproxy.cfg
```

* 启动HAProxy
```
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
```

* 开放防火墙端口
```
firewall-cmd --add-service=http --zone=internal --permanent # web services hosted on worker nodes
firewall-cmd --add-service=http --zone=external --permanent # web services hosted on worker nodes
firewall-cmd --add-service=https --zone=internal --permanent # web services hosted on worker nodes
firewall-cmd --add-service=https --zone=external --permanent # web services hosted on worker nodes
firewall-cmd --add-port=9000/tcp --zone=internal --permanent # HAProxy Stats
firewall-cmd --add-port=9000/tcp --zone=external --permanent # HAProxy Stats
firewall-cmd --reload
```

* 验证HA，浏览器访问https:192.168.1.1:9000/stats
![044d1c75861e15c1846bb1d9adf632fd.png](en-resource://database/697:1)

可以看到队列状态不是UP，是因为还没有配置，属于正常情况

### 1.6 邮件服务器

* 运行容器版的mailhog邮件服务器，为Helix安装时使用
```
#添加helm repo
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo update

#安装MailHog helm chart
helm install mailhog codecentric/mailhog -n email --create-namespace --set service.type=NodePort

#验证邮件服务创建成功（STATUS=Running）
kubectl -n email get pod
```

* 查询服务端口信息
```
node_ip=$(kubectl get nodes -o=jsonpath='{.items[0].status.addresses[0].address}')
web_port=$(kubectl --namespace email get svc mailhog -o=jsonpath="{.spec.ports[?(@.name=='http')].nodePort}")
smtp_port=$(kubectl --namespace email get svc mailhog -o=jsonpath="{.spec.ports[?(@.name=='tcp-smtp')].nodePort}")

echo "MailHog Web UI at http://$node_ip:$web_port"
echo "MailHog SMTP port at $node_ip:$smtp_port"


MailHog Web UI at http://192.168.1.200:31532
MailHog SMTP port at 192.168.1.200:32354
```
说明邮件控制台是http://192.168.1.200:31532，发送接口是192.168.1.200:32354

* 修改HAProxy配置文件，调整mailhog的端口为SMTP port输出值
```
vi /etc/haproxy/haproxy.cfg
```
backend mailhog
  mode tcp
    balance     leastconn
    server helix-k8s-worker01 192.168.1.200:**32354** check

* 重启HAProxy服务，使变更生效
```
systemctl restart haproxy
```

* 查看HAProxy控制台，验证mailhog运行

* 添加放开防火墙端口
```
firewall-cmd --add-port=25/tcp --zone=internal --permanent
firewall-cmd --add-port=31532/tcp --zone=internal --permanent
firewall-cmd --add-port=32354/tcp --zone=internal --permanent
firewall-cmd --reload
```

* 测试发邮件
```
dnf install epel-release -y 
dnf install swaks -y
# send email using swaks
swaks -f host-test@me -t local@me -s $node_ip -p $smtp_port --body "this is a test" --header "Subject: host validation via port: $smtp_port"
swaks -f host-test@me -t local@me -s 192.168.1.1 -p 25 --body "this is a test" --header "Subject: host validation via port: 25"
```
* 浏览器登录邮件控制台 https://192.168.1.200:31532，可以查看到两封邮件，分别发送给原端口和HAProxy代理的25端口
![58c4cf54dae22f61db0f730a1b449302.png](en-resource://database/699:1)


### 1.7 Ingress
Helix从24.3开始支持两种类型的Kubernetes反向代理和负载均衡

* NGINX Open Source Ingress Controller
* F5 NGINX Plus Ingress Controller

本文档中采用的是第一种。详细的介绍与安装步骤请参考文档[IngressController](https://docs.bmc.com/xwiki/bin/view/IT-Operations-Management/On-Premises-Deployment/BMC-Helix-IT-Operations-Management-Deployment/itomdeploy251/Deploying/Preparing-for-deployment/Deploying-and-configuring-the-NGINX-Open-Source-Ingress-Controller/)

* 删除旧的ingress-nginx命名空间

```
kubectl delete ds -n ingress-nginx ingress-nginx-controller
kubectl -n ingress-nginx delete svc ingress-nginx-controller-admission
kubectl delete clusterrole ingress-nginxkubectl delete ClusterRoleBinding ingress-nginx
kubectl delete IngressClass nginx
kubectl delete ValidatingWebhookConfiguration ingress-nginx-admission
kubectl delete ns ingress-nginx
```

* 下载对应kubernetes版本的[NGINX Ingress Controller](https://docs.bmc.com/xwiki/bin/view/IT-Operations-Management/On-Premises-Deployment/BMC-Helix-IT-Operations-Management-Deployment/itomdeploy251/Planning/System-requirements/) 

```
dnf install wget -y
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.4/deploy/static/provider/cloud/deploy.yaml
```

* 编辑修改下载的deploy.yaml文件

    Change the kind field of the ingress-nginx-controller from **Deployment** to **DaemonSet**
    Under kind: **Daemonset**, change the spec.**strategy** field to spec.**updateStrategy**
    Under kind: **Daemonset**, locate **securityContext**, and then set the value of the flag **allowPrivilegeEscalation** as **true**

*  部署deploy.yaml
```
kubectl create ns ingress-nginx
kubectl apply -f deploy.yaml
```

* 等待Ingress Nginx创建结束
```
kubectl -n ingress-nginx get all

NAME                                       READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-4fdv9   0/1     Completed   0          111s
pod/ingress-nginx-admission-patch-tdxjj    0/1     Completed   0          111s
pod/ingress-nginx-controller-dwxv7         1/1     Running     0          111s
pod/ingress-nginx-controller-kw2qt         1/1     Running     0          111s
pod/ingress-nginx-controller-llgpq         1/1     Running     0          111s
pod/ingress-nginx-controller-sbwrb         1/1     Running     0          111s

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.43.144.239   <pending>     80:30468/TCP,443:31019/TCP   112s
service/ingress-nginx-controller-admission   ClusterIP      10.43.137.171   <none>        443/TCP                      112s

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/ingress-nginx-controller   4         4         4       4            4           kubernetes.io/os=linux   111s

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           35s        112s
job.batch/ingress-nginx-admission-patch    Complete   1/1           35s        112s
```

* 创建secret my-tls-secret
```
kubectl create secret tls my-tls-secret --cert=/root/openssl/bmc.local.crt --key=/root/openssl/bmc.local.key -n ingress-nginx
```

* 修改daemonset，将缺省SSL证书指向my-tls-secret
```
kubectl edit daemonset ingress-nginx-controller -n ingress-nginx
```
修改后的配置如下：
![2eb13f3420d6b7d5783b23c1e2d8cbc3.png](en-resource://database/701:1)

* 修改ngress-nginx-controller
```
kubectl edit cm ingress-nginx-controller -n ingress-nginx
```

* 在data下增加如下内容：

```
  enable-underscores-in-headers: "true"  
  proxy-body-size: 250m  
  server-name-hash-bucket-size: "1024"  
  ssl-redirect: "false"  
  use-forwarded-headers: "true" 
  worker-processes: "40" 
  allow-snippet-annotations: "true"
```

* 修改后如下图：

![805725f82334d5b4441f7f6530ed99ea.png](en-resource://database/703:1)

* 重启daemonset
```
kubectl -n ingress-nginx rollout restart ds ingress-nginx-controller
```

* 等待pod重启完成
```
kubectl -n ingress-nginx get pod
```

* 验证采用了新的Ingress Controller版本
```
kubectl -n ingress-nginx describe <pod name> | grep -i image
```
验证结果如下图，
```
[root@helix-svc openssl]# kubectl -n ingress-nginx get pod
NAME                                   READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-4fdv9   0/1     Completed   0          46m
ingress-nginx-admission-patch-tdxjj    0/1     Completed   0          46m
ingress-nginx-controller-2q2vg         1/1     Running     0          13s
ingress-nginx-controller-8h92b         1/1     Running     0          98s
ingress-nginx-controller-wrmq5         1/1     Running     0          66s
ingress-nginx-controller-zt5wg         1/1     Running     0          35s
[root@helix-svc openssl]# kubectl -n ingress-nginx describe ingress-nginx-controller-2q2vg | grep -i image
error: the server doesn't have a resource type "ingress-nginx-controller-2q2vg"
[root@helix-svc openssl]# kubectl -n ingress-nginx describe pod ingress-nginx-controller-2q2vg | grep -i image
    Image:           registry.k8s.io/ingress-nginx/controller:v1.11.4@sha256:981a97d78bee3109c0b149946c07989f8f1478a9265031d2d23dea839ba05b52
    Image ID:        docker-pullable://registry.k8s.io/ingress-nginx/controller@sha256:981a97d78bee3109c0b149946c07989f8f1478a9265031d2d23dea839ba05b52
  Normal  Pulled     50s   kubelet                   Container image "registry.k8s.io/ingress-nginx/controller:v1.11.4@sha256:981a97d78bee3109c0b149946c07989f8f1478a9265031d2d23dea839ba05b52" already present on machine
```
* 删除Validating Webhook Configuration，否则可能会妨碍ingress对象的发布
```
kubectl delete job ingress-nginx-admission-create ingress-nginx-admission-patch -n ingress-nginx --ignore-not-found=true
kubectl -n ingress-nginx delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```
* 更新服务ingress-nginx-controller将external IP指向服务在均衡器的地址
```
kubectl patch service/ingress-nginx-controller -n ingress-nginx -p '{"spec":{"externalIPs":["192.168.1.1"]}}'
```

* 验证ingress-nginx-controller被成功更改
```
kubectl -n ingress-nginx get service

NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.43.144.239   192.168.1.1   80:30468/TCP,443:31019/TCP   55m
ingress-nginx-controller-admission   ClusterIP      10.43.137.171   <none>        443/TCP                      55m
```

* 根据返回的443映射端口号（本例中是31019），更改HAProxy的配置
```
vi /etc/haproxy/haproxy.cfg
```

* 更改后的效果截图
![b027fdec6d6e0ca6899128374a7f98b6.png](en-resource://database/705:1)

* 重启HAProxy服务，启用新配置
```
systemctl restart haproxy
```

* 浏览器登录HAProxy控制台，验证结果
![69e6aece587f5bbeafccb163ff9585b6.png](en-resource://database/707:1)






## 2 安装Helix Dashboard

## 3 安装Helix Discovery
## 4 安装其他ITOM组件
