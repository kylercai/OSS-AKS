# Note: The included sample manifest files use Azure Region eastus2.

## Steps:
### 1. create AKS with multiple AZ support. 
Refer to https://learn.microsoft.com/en-us/azure/aks/availability-zones#create-an-aks-cluster-across-availability-zones

### 2. install the RocketMQ NameServer component into AKS (with 1 replica)
```
kubectl apply -f nameserver.yml
```

### 3. apply configmap into AKS for RocketMQ brokers (2m-2s)
```
kubectl apply -f brokers-configmap.yml
```

### 4. install storage class to support zone redundant persistent volume into AKS
```
kubectl apply -f storageclasszrs.yml
```
_you can refer to https://learn.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes for details_

### 5. install brokers (broker-a & broker-b) master nodes
```
kubectl apply -f brokers-masters.yml
```
_The broker-a master will be placed in zone-1, and the broker-b master will be placed in zone-2_

### 6. install brokers (broker-a & broker-b) slave nodes
```
kubectl apply -f brokers-slaves.yml
```
_The slaves will be placed in zone-3_

### 7. install RocketMQ console into AKS
```
kubectl apply -f console.yml
```

### 8. check your installation
```
kubectl get pods -o wide
```
**you should see 6 pods: 1 nameserver pod, 2 broker-a pods, 2 broker-b pods, 1 console pod.**
**You will see like below:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/get-pods.jpg)

### 9. access RocketMQ console
```
kubectl get service
```
**get the console service IP:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/get-services.jpg) <br>

**access the IP:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/console-1.jpg)
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/console-2.jpg)

### 10. test RocketMQ
**get into broker-a-0 pod:**
```
kubectl exec -it broker-a-0 -- /bin/bash
```
**run below commands to test RocketMQ**
```
export NAMESRV_ADDR=mq-ns.default.svc.cluster.local:9876
./tools.sh org.apache.rocketmq.example.quickstart.Producer
```
**you will see below output:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/msg-producer.jpg)