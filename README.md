# CKAD-NOTE
## Common Comand
kubectl create deploy D	EPOLY_NAME –image=IMAGE_NAME
kubectl run pod POD_NAME –image=MY_IMAGE –dry-run=client -o yaml >> file.yaml
kubectl create namespace NAMESPACE_NAME
kubectl port-forward fwnginx 8080 :80
kubectl exec -it POD_NAME – CMD1 CMD2

## Security context
kubectl explain pod.spec.securityContext
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec: 
  securityContext: # Pod security Context
    runAsUser: 1000        <---
    runAsGroup: 3000       <---
    fsGroup: 2000          <---
  containers:
    securityContext: #Container Security context 
      allowPrivilegeEscalation: false    <---
```
## Job

```bash
kubectl create job mynewjob --image=busybox --dry-run=client -o yaml -- sleep 5 > mynewJob.yaml
```
### Parallelism/completion
```yaml
kind: Job
spec:
  parallelism: 1
  completions: 3
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: mynewjob
spec:
  ttlSecondsAfterFinished: 60  <---
  completions: 3               <---
  template:
    metadata:
      creationTimestamp: null  <---
    spec:
      containers:
      - command:
        - sleep
        - "5"
        image: busybox
        name: mynewjob
        resources: {}
      restartPolicy: Never
status: {}
```
### Cronjobs
```bash
kubect create cronjobs runmeat --image=busybox --schedsule="*/1 * * * *" -- echo greetings drom the cluster
```

## Managing Ressources

```yaml
apiVersio: v1
kind: Pod
metadata:
  name: RessourceManager
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: password"
    ressources:          <---
      requests:          <---
        memory: "64Mi"   <---
        cpu: "250m"      <---
      limits:            <---
        memory: "128Mi"  <---
        cpu: "500m"      <---
  - name: wp
    image: wordpress
    ressources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Clean up ressource
```bash
kubectl delete all --all
```

## Deployeemnt

Pour montré toute les ressources d'une app
```bash
kubectl get all --selector app=my-deployement-name
```

Pour update un paramètre tesl que l'image
```bash
kubectl set image deploy my-deployement-name nginx=nginx:1.17
```

## Label  key=value

Create deployment and affect label
```bash
kubectl create deploy bluelabel --image=nginx
kubectl create label deployment bluelabel state=demo
```

Voir les labels
```bash
kubectl get all --show-labels
```

Filtrer par label
```bash
kubectl get all --selector state=demo
```

Remove label   'key-'
```bash
kubectl label deployment bluelabel state-
```

## Update Strategy
Recreate : delet all and recreate (cannot run differente versions of an application like DB)
RollingUpdate : One by one

```bash
kubectl rollout history #Pour voir l'historique d'update
kubectl rollout histroy deployment rolling-nginx --revision=1
kubectl rollout undo #pour rollback
#For exemple :
 
```

```bash
kubectl rollout histroy deployment rolling-nginx
deployment.apps/rolling-nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

### Spec
- maxUnavaible
- maxSurge 

## Deployement

### DaemonSet
One pod on each node

```yaml
apiVersion: apps/v1
kind: DaemonSet
...
```

## Expose

```bash
kubectl expose deploy nginx --port=80 # => create an service (svc)
kubectl get endpoints
kubectl get svc
kubectl edit svc nginx # Modify svc type as Nodeport for example 
```

## Ingres

Create ingress
```bash
                                      #refer to the port in the docker ?
kubectl create ingress rolling-nginx --rule="/=rolling-nginx:80" --rule="/hello=newdep:8080"
kubectl describe ingress rolling-nginx
```

## Network Policy

example [ici](https://github.com/sandervanvugt/ckad/blob/master/nwpolicy-complete-example.yaml)

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx   #--> La policy sapplique sur l'app nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true" #--> le pod qui veux target devra avoir le lable access: "true"
```

# Volume
TOUJOURS créer :
  - PVC
  - PV 
## PV

volume type:
 - EmptyDir
 - hostpath
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: morevol2
spec:
  containers:
  - name: centos1
    image: centos:7
    command:
      - sleep
      - "3600" 
    volumeMounts:
      - mountPath: /centos1
        name: test
  - name: centos2
    image: centos:7
    command:
      - sleep
      - "3600"
    volumeMounts:
      - mountPath: /centos2
        name: test
  volumes: 
    - name: test
      emptyDir: {}
```

## PVC

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
---
kind: Pod
apiVersion: v1
metadata:
   name: nginx-pvc-pod
spec:
  volumes:
    - name: site-storage
      persistentVolumeClaim:
        claimName: nginx-pvc
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: webserver
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: site-storage
```

# ConfigMap
For :
  - En var
  - conf file   (on peux mettre un directory dedans)
  - command line

From:
  - --from-env-file  
  - --frome-literal
  - kubectl set env -- from=configmap/mycm deply/myapp

```bash
kubectl create cm mydbvars--from-en-file=myvarfile
kubectl create deploy mydb --image=mariadb --replicas=3
kubectl set env deploy mydb --from=configmap/mydbvars
```