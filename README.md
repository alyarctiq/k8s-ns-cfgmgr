# k8s-ns-cfgmgr
Simple (non informer) based operator to poll git repo for yaml files of Kind (Deployment and Service) and maintain state in a namespace. 


Usage:
```
-- demo with hipster store appl -- 

git clone repo
cd deploy/

# Create Namesapce
$ kubectl create ns hipster
namespace/hipster created

# Apply RBAC
$ kubectl apply -f rbac-r.yaml -n hipster
serviceaccount/k8s-ns-cfgmgr created
role.rbac.authorization.k8s.io/k8s-ns-cfgmgr created
rolebinding.rbac.authorization.k8s.io/k8s-ns-cfgmgr created

# Deploy Op
$ kubectl apply -f go-op-rc.yaml  -n hipster
replicationcontroller/k8s-ns-cfgmgr created

$ kubectl get pods -n hipster
NAME                  READY   STATUS              RESTARTS   AGE
k8s-ns-cfgmgr-ppcfk   0/1     ContainerCreating   0          3s
```

Validate
```
# Tail Logs
$ kubectl logs -f k8s-ns-cfgmgr-8vvkc  -n hipster

2020/04/11 14:26:07 OS ENV NS:  hipster
2020/04/11 14:26:07 OS ENV URL:  https://github.com/alyarctiq/k8s-ns-cfgmgr.git
2020/04/11 14:26:07 Cloning Git Repo

git pull https://github.com/alyarctiq/k8s-ns-cfgmgr.git /tmp/repo --recursive

2020/04/11 14:26:08 Search Path: Found /tmp/repo/yamls
2020/04/11 14:26:08 Starting Watch Loop...
2020/04/11 14:26:08 Loading Yaml Files: /tmp/repo/yamls/hipster1of2.yaml
2020/04/11 14:26:08 Loading Yaml Files: /tmp/repo/yamls/hipster2of2.yaml
2020/04/11 14:26:08 Loading Master Files:  /tmp/master.yaml
```

Reconciling Namespace Objects
```
# Deployments
2020/04/11 14:21:54 Repairing Missing Deployment: emailservice
Created deployment "emailservice".
2020/04/11 14:21:54 Repairing Missing Deployment: checkoutservice
Created deployment "checkoutservice".
2020/04/11 14:21:55 Repairing Missing Deployment: recommendationservice
Created deployment "recommendationservice".
2020/04/11 14:21:55 Repairing Missing Deployment: frontend
Created deployment "frontend".
2020/04/11 14:21:55 Repairing Missing Deployment: paymentservice
Created deployment "paymentservice".
2020/04/11 14:21:55 Repairing Missing Deployment: productcatalogservice
Created deployment "productcatalogservice".
2020/04/11 14:21:55 Repairing Missing Deployment: cartservice
Created deployment "cartservice".
2020/04/11 14:21:55 Repairing Missing Deployment: loadgenerator
Created deployment "loadgenerator".
....

# Services
2020/04/11 14:21:56 Repairing Missing Service: emailservice
Created Service "emailservice".
2020/04/11 14:21:57 Repairing Missing Service: checkoutservice
Created Service "checkoutservice".
2020/04/11 14:21:58 Repairing Missing Service: recommendationservice
Created Service "recommendationservice".
2020/04/11 14:21:59 Repairing Missing Service: frontend
Created Service "frontend".
2020/04/11 14:22:00 Repairing Missing Service: frontend-external
Created Service "frontend-external".
2020/04/11 14:22:00 Repairing Missing Service: paymentservice
Created Service "paymentservice".
.....

$ kubectl get pods -n hipster
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-5897f58b66-4brv2               0/1     Running   0          50s
cartservice-5d679b9449-c6w49             1/1     Running   0          52s
checkoutservice-6f56ff8674-skptd         1/1     Running   0          53s
currencyservice-f58f7f5d4-vl4nv          1/1     Running   0          52s
emailservice-d66dc8fbb-4gkzw             0/1     Running   2          53s
frontend-5bd68756d4-x72p6                1/1     Running   0          53s
k8s-ns-cfgmgr-jl589                      1/1     Running   0          91s
loadgenerator-6655f746fb-rnq8g           1/1     Running   0          52s
paymentservice-794c8b9ccd-5tdh9          1/1     Running   0          53s
productcatalogservice-6446f67666-zlb76   1/1     Running   0          53s
recommendationservice-654cb4cdb4-b86qg   0/1     Running   1          53s
redis-cart-65bf66b8fd-8b7x6              1/1     Running   0          51s
shippingservice-849db7fbb-x8lv4          1/1     Running   0          51s
```


Testing
```
Action: Manually delete some deplyoment or services from cluster
Outcome: Operator finds delta between repo and cluster state and reconcilies changes.

Action: Adjust yaml files in Git repo
Outcome: Operator finds delta between cluster and repo state and reconcilies changes.
```

Reconcilie when a change is made to the repo. (eg yaml file is updated/deleted)
```
...
Delete Dep From Cluster: adservice
Deleted deployment.
Delete Dep From Cluster: cartservice
Deleted deployment.
Delete Dep From Cluster: currencyservice
Deleted deployment.
Delete Dep From Cluster: loadgenerator
Deleted deployment.
Delete Dep From Cluster: redis-cart
Deleted deployment.
Delete Dep From Cluster: shippingservice
Deleted deployment.
Deleting Svc: adservice
Delete Svc From Cluster: adservice
Deleting Svc: cartservice
Delete Svc From Cluster: cartservice
Deleting Svc: currencyservice
Delete Svc From Cluster: currencyservice
Deleting Svc: redis-cart
Delete Svc From Cluster: redis-cart
Deleting Svc: shippingservice
Delete Svc From Cluster: shippingservice
```


Customize
```
git clone repo
cd deploy/

edit go-op-rc.yaml

# edit ENV var
          env:
            # URL of your repo here (Public repo only for now)
          - name: URL
            value: "https://github.com/alyarctiq/k8s-ns-cfgmgr.git"
            
            # Folder where yaml files are located ("/path"). If in root use "."
          - name: FOLDER
            value: "/yamls"
            
            # Namespace to deploy into. Usually the same as operator. 
            # If you require seperates operator and app namespaces, see deploy folder for more rbac and repliction deployment examples. 
          - name: NAMESPACE
            value: "hipster"    

# Create Namesapce
$ kubectl create ns <namespace>


# Apply RBAC
$ kubectl apply -f rbac-r.yaml -n <namespace>
serviceaccount/k8s-ns-cfgmgr created
role.rbac.authorization.k8s.io/k8s-ns-cfgmgr created
rolebinding.rbac.authorization.k8s.io/k8s-ns-cfgmgr created

# Deploy Op
$ kubectl apply -f go-op-rc.yaml  -n <namespace>
replicationcontroller/k8s-ns-cfgmgr created     
      
```

Clean Up
```
Delete namespace

$ kubectl delete ns hipster
```

Build
```
git clone repo
go mod tidy
code
go build
adjust Dockerfile
adjust replication yaml

enjoy
```

