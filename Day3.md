# Day 3 
---
#### Create namespace 

```bash 
kubectl create namespace lab 
```

#### Create deployment with 3 replicas 
```bash
kubectl create deployment nginx --image=nginx --replicas=3 -n=lab
```
#### Expose the deployment on port 80 
```bash
kubectl expose deployment nginx --type=NodePort --port=80 --name=depservice -n=lab
```
#### Create a CronJob for listing the EndPoints

1. Create a **serviceaccount** cronjob-sa  
```bash 
kubectl create serviceaccount cronjob-sa -n=lab
```
2. Create a Role that allows listing all the services and endpoints
    
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: cronjob-role
  namespace: lab
rules:
- apiGroups:
  - ""
  resources: ["services", "endpoints"]
  verbs: ["get", "watch", "list"]
```

```bash 
kubectl apply -f role.yaml
```
3. Link the Role with the created SA 
```bash 
kubectl create rolebinding cronjob-sa-role --role=cronjob-role --serviceaccount=lab:cronjob-sa --namespace=lab
```
4. Create a CronJob that lists the endpoints in that namespace every minute and paste the output for the first pod created

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: list-endpoint
  namespace: lab
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: cronjob-sa
          containers:
          - name: cronjob-list
            image: bitnami/kubectl:latest
            resources: {}
            command:
            - kubectl
            - get
            - endpoints
          restartPolicy: OnFailure
```
```bash
kubectl apply -f cronjob.yaml
```

```bash 
kubectl logs pods/list-endpoint-27902359-dc5pg --namespace=lab
```
- Output 
```bash
  NAME         ENDPOINTS                                   AGE
depservice   172.17.0.5:80,172.17.0.6:80,172.17.0.7:80   18m
```
5. After listing try to delete the 3 nginx pods ? again try to view the logs for the newly created pod for that cronJob what do you think happened ? 

> after deleting the pods replicaset controller triggers that the pods is deleted and created new pods immediately.

> - by running get logs again we found that the endpoints changed because the new pods is created with diffrent ip address .

```bash 
kubectl logs pods/list-endpoint-27902364-wxs5l --namespace=lab

NAME         ENDPOINTS                                   AGE
depservice   172.17.0.5:80,172.17.0.6:80,172.17.0.8:80   23m
```

