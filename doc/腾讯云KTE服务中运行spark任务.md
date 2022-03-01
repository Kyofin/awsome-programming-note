# 腾讯云kte服务
申请托管-节点数3-2c4G-公网流量按量

wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz" \
    && tar -zxvf /opt/jdk-8u141-linux-x64.tar.gz \
    && rm -f /opt/jdk-8u141-linux-x64.tar.gz

wget https://archive.apache.org/dist/spark/spark-3.1.3/spark-3.1.3-bin-hadoop2.7.tgz

```
export JAVA_HOME=/opt/jdk1.8.0_141
export PATH=$PATH:$JAVA_HOME/bin

```

kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default

kubectl cluster-info

docker pull apache/spark-py:v3.1.3

spark 包用kubernetes-client-4.12.0 只支持K8s 1.18.0

./bin/spark-submit \
    --master k8s://https://169.254.128.8:60002 \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=3 \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
    --conf spark.kubernetes.container.image=apache/spark-py:v3.1.3 \
    local:///opt/spark/examples/jars/spark-examples_2.12-3.1.3.jar  1000000

## 资源清理
正常运行完driver的pod会显示completed，但仍能看到，executor的pod会自动消失。

## 报错信息
### 账号没权限
```
Caused by: io.fabric8.kubernetes.client.KubernetesClientException: Failure executing: GET at: https://kubernetes.default.svc/api/v1/namespaces/default/pods/spark-pi-12c5d77f2447f0ca-driver. Message: Forbidden!Configured service account doesn't have access. Service account may have been revoked. pods "spark-pi-12c5d77f2447f0ca-driver" is forbidden: User "system:serviceaccount:default:default" cannot get resource "pods" in API group "" in the namespace "default".

```
增加配置： 
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark 


