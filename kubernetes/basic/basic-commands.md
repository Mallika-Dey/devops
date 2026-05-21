```bash
Pod
├── apiVersion  → Kubernetes API version
├── kind        → Type of object
├── metadata    → Information about object
│   ├── name
│   └── labels
└── spec        → Desired state/configuration
    └── containers
```

```bash
minikube start

kubectl run nginx --image=nginx

kubectl get pods

kubectl describe pod nginx

kubectl get pods -o wide

kubectl delete pod nginx

kubectl apply -f pod-defi.yml 

kubectl create -f pod-defi.yml 

kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```

```bash
kubectl get replicaset

kubectl describe replicaset new-replica-set 

kubectl delete pods new-pod

# create replicaset
kubectl create -f replicaset.yaml

kubectl delete replicaset replicaset-1 replicaset-2
# scale replicaset
kubectl scale rs myreplicaset --replicas=5
```