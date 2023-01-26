### Day 5 Lab

1-The DevOps team would like to get the list of all Namespaces in the cluster. Get the list and save it to /opt/namespaces.n
```bash
kubectl get namespaces -A
NAME              STATUS   AGE
default           Active   30d
kube-node-lease   Active   30d
kube-public       Active   30d
kube-system       Active   30d
```
---

2- create  ServiceAccount named neptune-sa-v2 in Namespace neptune.
```bash
kubectl create namespace neptune
kubectl create serviceaccount neptune-sa-v2 -n neptune
```
------------------------------
3- Create a new ConfigMap named cm-3392845. Use the spec given on the below.

ConfigName Name: cm-3392845

Data: DB_NAME=SQL3322

Data: DB_HOST=sql322.mycompany.com

Data: DB_PORT=3306

```bash
kubectl create configmap cm-3392845 --from-file config -oyaml > createConfig.yaml
```



-------------------------------
4-Team Pluto needs a new cluster internal Service. Create a ClusterIP Service named project-plt-6cc-svc in Namespace pluto. This Service should expose a single Pod named project-plt-6cc-api of image nginx:1.17.3-alpine, create that Pod as well. The Pod should be identified by label project: plt-6cc-api. The Service should use tcp port redirection of 3333:80.

1)
```bash
 k create namespace pluto
 ``` 
2) 
- run command :
```bash
 kubectl run project-plt-6cc-api --image=nginx:1.17.3-alpine -oyaml --dry-run=client > pod.yaml 
 ```
- To add label
```bash
-vi pod.yaml 
-k apply -f pod.yaml
```  
3) 
```bash
- kubectl create service clusterip project-plt-6cc-api --tcp=3333:80 --dry-run=client -o yaml > serviceClusterIP.yaml
```

- To edit the labels
```bash
vi serviceCluserIp.yaml
kubectl apply -f serviceClusterIP.yaml -n pluto 
```

---------------------------------------------------------------
5-
Create a new PersistentVolume named earth-project-earthflower-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace earth named earth-project-earthflower-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment project-earthflower in Namespace earth which mounts that volume at /tmp/project-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.


1) Create pv
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: earth-project-earthflower-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /volumes/Data
                         
```

2) Create pvc 
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: earth-project-earthflower-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi 

k apply -f pvc.yaml -n earth-project-earthflower-pvc
```
- Create Deployment 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deployment
  name: deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deployment
    spec:
      volumes:
        - name: httpd
          persistentVolumeClaim:
            claimName: earth-project-earthflower-pvc
      containers:
      - image: httpd:2.4.41-alpine
        name: httpd
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: httpd
        resources: {}
status: {}
```
---

