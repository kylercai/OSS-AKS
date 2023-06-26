# Note: The included sample manifest files use Azure Region eastus2.

## Steps:
### 1. create AKS with multiple AZ support. Refer to https://learn.microsoft.com/en-us/azure/aks/availability-zones#create-an-aks-cluster-across-availability-zones

### 2. install the RocketMQ NameServer component into AKS (with 1 replica)
kubectl apply -f ./nameserver.yml

### 3. apply configmap into AKS for RocketMQ brokers (2m-2s)
kubectl apply -f brokers-configmap.yml

### 4. install storage class to support zone redundant persistent volume into AKS. Refer to https://learn.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes
kubectl apply -f ./storageclass.yml

### 5. install brokers (broker-a & broker-b) master nodes. The broker-a master will be placed in zone-1, and the broker-b master will be placed in zone-2
kubectl apply -f brokers-masters.yml

### 6. install brokers (broker-a & broker-b) slave nodes. The slaves will be placed in zone-3
kubectl apply -f brokers-slaves.yml