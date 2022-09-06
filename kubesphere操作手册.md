### 多结点安装

##### 准备Linux主机

| 主机名    | 系统           | 角色           |
|:------ | ------------ | ------------ |
| master | Ubuntu 18.04 | master, etcd |
| node1  | Ubuntu 18.04 | worker       |
| node2  | Ubuntu 18.04 | worker       |

##### 下载KubeKey

```bash
# 设置区域
export KKZONE=cn

# 下载KubeKey
curl -sfL https://get-kk.kubesphere.io | VERSION=v2.0.0 sh -

# 添加可执行权限
chmod +x kk
```

##### 创建集群

```bash
# 生成集群配置文件(安装有KubeSphere的Kubernetes集群)
./kk create config --with-kubernetes v1.21.5 --with-kubesphere v3.2.1

# 修改集群配置文件
vi config-sample.yaml

# 创建集群
./kk create cluster -f config-sample.yaml

# 查看安装日志
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

##### 添加节点

```bash
# 修改集群配置文件
vi config-sample.yaml

# 添加节点
./kk add nodes -f config-sample.yaml
```

### 配置调整

##### NodePort端口范围

- 方法1：安装集群前修改配置文件
  
  ```vim
  kubernetes:
    version: v1.21.5
    clusterName: cluster.local
    apiserverArgs:
    - service-node-port-range=60000-63880
  ```

- 方法2：安装后修改Kubernetes配置文件
  
  ```bash
  # master修改kube-apiserver.yaml
  vi /etc/kubernetes/manifests/kube-apiserver.yaml
  ```
  
  ```vim
  spec:
    containers:
    - command:
      - kube-apiserver
      - --service-node-port-range=60000-63880
  ```
  
  ```bash
  systemctl daemon-reload
  systemctl restart kubelet
  ```

### 中间件安装

##### Mysql集群(RadonDB v2.2.0)

- 添加应用仓库
  名称：radondb-mysql-operator
  URL：https://radondb.github.io/radondb-mysql-kubernetes/

- 部署应用mysql-operator

- 部署mysql-cluster

```bash
# 获取配置
wget https://github.com/radondb/radondb-mysql-kubernetes/releases/download/v2.2.0/mysql_v1alpha1_mysqlcluster_mysql8.yaml

# 修改配置
vim mysql_v1alpha1_mysqlcluster_mysql8.yaml
# name: sample => name: mysql-cluster

# 应用配置
kubectl apply -f mysql_v1alpha1_mysqlcluster_mysql8.yaml --namespace=middleware
```

- 添加test用户

在项目 middleware 中，创建保密字典 mysql-cluster-password (类型默认，添加数据 键:pwdForTest 值:test用户密码)

```bash
# 获取配置
wget https://raw.githubusercontent.com/radondb/radondb-mysql-kubernetes/main/config/samples/mysql_v1alpha1_mysqluser.yaml

# 修改配置
vi mysql_v1alpha1_mysqluser.yaml
# name: sample-user-cr => name: mysql-cluster-user-cr
# user: sample_user => user: test
# privileges: SELECT => privileges: ALL
# clusterName: sample => clusterName: mysql-cluster
# nameSpace: default => nameSpace: middleware
# secretName: sample-user-password => secretName: mysql-cluster-password
# secretKey: pwdForSample => secretKey: pwdForTest

# 应用配置
kubectl apply -f mysql_v1alpha1_mysqluser.yaml --namespace=middleware
```

- 测试

```bash
# kubectl测试访问
kubectl exec -it mysql-cluster-mysql-0 -n middleware -- mysql -u test -p
```

# 同集群使用service_name访问

```bash
mysql -h mysql-cluster-leader.middleware -u luke -p
```

##### Redis主副本集群(Bitnami v16.8.5)

- 添加应用仓库
  名称：bitnami
  URL：https://charts.bitnami.com/bitnami

- 部署应用redis-cluster

##### Elasticsearch集群(ECK)

- 部署operator到middleware

```bash
# 安装CRD
kubectl create -f https://download.elastic.co/downloads/eck/2.1.0/crds.yaml

# 获取operator配置
wget https://download.elastic.co/downloads/eck/2.1.0/operator.yaml

# 修改operator配置
vi operator.yaml
# 删除elastic-system命名空间的创建
# 把默认的elastic-system改为middleware
# :%s/elastic-system/middleware/g

# 部署operator
kubectl create -f operator.yaml 
```

- 部署Elasticsearch Cluster

```bash
# 创建cluster配置
cat <<EOF >> cluster.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-cluster
  namespace: middleware
spec:
  version: 8.1.2
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
EOF

# 部署cluster
kubectl create -f cluster.yaml

# 访问测试
curl -u "elastic:xxxxxxxxxxxxxxx" -k "https://10.0.0.xxx:6xxxx"
```

- 部署Kibana实例

```bash
# 创建cluster配置
cat <<EOF >> kibana.yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
  namespace: middleware
spec:
  version: 8.1.2
  count: 1
  elasticsearchRef:
    name: elasticsearch-cluster
    namespace: middleware
EOF

# 部署cluster
kubectl create -f kibana.yaml
```


### 常见问题

##### 强制删除CRD

```bash
kubectl patch crd/CRD名 -p '{"metadata":{"finalizers":[]}}' --type=merge
```

