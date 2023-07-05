# Note: 
**This example shows how to run RocketMQ with multiple AZ and broker auto failover on Azure Kubernetes Service.**<br>
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
**get the console service IP:** 
```
kubectl get service
```
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
_*The broker instances will be scheduled/distributed across multiple zones._<br>
_*The maximum instance number difference between zones is 1 (defined by topologySpreadConstraints.maxSkew : 1 in the raft-broker0.yml)_<br>
```
kubectl get pods -o wide
```
**you should see 7 pods: 3 nameserver/controller pods, 3 broker pods, 1 console pod:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/03-step10-1-check-your-installation.jpg)
**And the broker instances are scheduled across multiple zones:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/03-step10-3-nodes-across-AZ.jpg)
**on the RocketMQ console, you will see the broker has 1 master instance and the rest are slave instances**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/03-step10-2-console-master-slaves.jpg)


### 11. Send a test message to broker on console
_On the RocketMQ console, go to the Topic sheet, and create a topic for example "DemoTopic". Then by clicking the "SEND MESSAGE" button of the topic, send a message for testing like below:_
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/04-step11-1-console-send-test-message.jpg)
_Go to the Message sheet, by selecting topic as "DemoTopic" and clicking "SEARCH" button, you will see the message you just sent:_
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/04-step11-2-console-query-test-message.jpg)

### 12. Verify the auto failover for broker instances
**To simulate broker master crashing, go to the AKS console to delete the broker master instance pod:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/05-step12-1-delete-broker-master-node-pod.jpg)
**after the broker master instance pod terminates, in the RocketMQ console you will see the master node is automatically switched over to another pod instance:** 
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/05-step12-2-master-auto-switched.jpg)
**Take actions below to verify the broker is still working after auto failover switching:**<br>
_*on the RocketMQ console, go to the Message sheet, and check the message we send above for test is still there._<br>
_*send a message to verify the broker is still working._<br>
_*because we run broker instances with StatefulSet, it will automatically recover the pod instance we killed above. Once the pod is recoved, it plays as slave role:_
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/05-step12-3-new-master-slaves.jpg)

### 13. Verify scale in for broker instances
```
kubectl scale --replicas=1 statefulset/broker0
```
**after the broker instances scale in, in the RocketMQ console you will see the broker makes sure there is a running master node:**
![](https://github.com/kylercai/OSS-AKS/blob/master/rocketmq-raft/06-step13-1-scale-in-result.jpg)
_*You can then send another test message to verify the broker is still working_

**End**