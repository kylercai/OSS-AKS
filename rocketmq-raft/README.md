# Note: The included sample manifest files use Azure Region eastus2.

## Steps:
### 1. create AKS with multiple AZ support. 
Refer to https://learn.microsoft.com/en-us/azure/aks/availability-zones#create-an-aks-cluster-across-availability-zones

### 2. apply configmap into AKS for RocketMQ NameServer and Controller (3 nodes)
```
kubectl apply -f nameserver-controller-configmap.yml
```


### 3. install the RocketMQ NameServer and Controller component into AKS (3 nodes)
```
kubectl apply -f nameserver-controller.yml
```

### 4. apply configmap into AKS for RocketMQ brokers with Controller mode enabled
```
kubectl apply -f raft-brokers-configmap.yml
```

### 5. install broker (broker name: broker0) with multiple AZ distributed. The broker instance number is defined by replicas in raft-broker0.yml
```
kubectl apply -f raft-broker0.yml
```
_The broker instances will be scheduled/distributed across multiple zones. The maximum instance number difference between zones is 1 (defined by topologySpreadConstraints.maxSkew : 1)_

### 6. install RocketMQ console into AKS
```
kubectl apply -f console.yml
```

### 7. scale out the broker instances. The number should always be an odd number
```
kubectl scale --replicas=3 statefulset/broker0
```

### 7. check your installation, and verify the broker instances are scheduled across multiple zones
```
kubectl get pods -o wide
```
**you should see 7 pods: 3 nameserver/controller pods, 3 broker pods, 1 console pod.**
**You will see like below:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/get-pods.jpg)
**You will see the broker instances are scheduled across multiple zones:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/get-pods.jpg)
**You will see the broker has 1 master instance and the rest are slave instances**
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

### 10. Send a test message to broker on console
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/msg-producer.jpg)

### 11. Verify the auto failover for broker instances
**get the AKS console to delete the broker master instance pod:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/msg-producer.jpg)
**after the broker master instance pod terminated, in the RocketMQ console you will see the master node is automatically switched over to another pod instance:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/msg-producer.jpg)
**because we run broker instances with StatefulSet, it will automatically recover the pod instance we killed above. Once the pod is recoved, it plays as slave role:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq/msg-producer.jpg)