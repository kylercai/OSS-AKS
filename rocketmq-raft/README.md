# Note: 
**The included sample manifest files use Azure Region eastus2.**<br>
**The included sample run RocketMQ Controller inside of NameServer.**

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

### 5. install broker (broker name: broker0) with multiple AZ. 
_*The broker instance number is defined by replicas in raft-broker0.yml._<br>
_*The number should always be an odd number._<br>
_*The broker instances will be scheduled/distributed across multiple zones._<br>
_*For now we run the broker with just 1 instance for starting up._<br>
```
kubectl apply -f raft-broker0.yml
```

### 6. install RocketMQ console into AKS
```
kubectl apply -f console.yml
```

### 7. check your installation
```
kubectl get pods -o wide
```
**you should see 5 pods: 3 nameserver/controller pods, 1 broker pods, 1 console pod.**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/01-step7-1-check-your-installation.jpg)

### 8. access RocketMQ console
```
kubectl get service
```
**get the console service IP:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/02-step8-1-get-console-service.jpg) <br>

**access the IP:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/02-step8-2-access-console.jpg)
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/02-step8-2-access-console-cluster.jpg)

### 9. scale out the broker instances. 
**The number should always be an odd number, for example 3**
```
kubectl scale --replicas=3 statefulset/broker0
```

### 10. check your installation, and verify the broker instances are scheduled across multiple zones
```
kubectl get pods -o wide
```
_The broker instances will be scheduled/distributed across multiple zones. The maximum instance number difference between zones is 1 (defined by topologySpreadConstraints.maxSkew : 1 in the raft-broker0.yml)_

**you should see 7 pods: 3 nameserver/controller pods, 3 broker pods, 1 console pod. And the broker instances are scheduled across multiple zones:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/get-pods.jpg)
**on the RocketMQ console, you will see the broker has 1 master instance and the rest are slave instances**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/get-pods.jpg)


### 11. Send a test message to broker on console
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/msg-producer.jpg)

### 12. Verify the auto failover for broker instances
**go to the AKS console to delete the broker master instance pod:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/msg-producer.jpg)
**after the broker master instance pod terminates, in the RocketMQ console you will see the master node is automatically switched over to another pod instance:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/msg-producer.jpg)
**check the message we send above for test is still there:**
![](
**send a message to verify the broker is still working:**
![](
**because we run broker instances with StatefulSet, it will automatically recover the pod instance we killed above. Once the pod is recoved, it plays as slave role:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/msg-producer.jpg)

### 13. Verify the scale in for broker instances
```
kubectl scale --replicas=1 statefulset/broker0
```
**after the broker instances scaled in, in the RocketMQ console you will see the broker instances are scaled in:**
![](
**on the RocketMQ console, you can see broker makes sure there is running master node:**
![](
**send a message to verify the broker is still working:**
![](