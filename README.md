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

# Secret

- docker-registry
- TLS
- generic

```bash
kubectl create secret SECRET_TYPE #(docker-registry, TLS, generic)
kubectl create secret tls my-tls-keys --cert=tls/my.crt --key=tls.key
kubectl create secret generic my-generic-pwd --from-litteral=password=verysecret
kubectl create secret generic my-ssh-key --from-file=ssh-private-key=.ssh/id_rsa
kubectl create secret generic my-secret-file --from-file=/my/file
```

After set env as you do it with env variable 
```bash
kubectl set env --from=configmap/myconfigmap --prefix=MYSQL_ deployment/myapp

kubectl create secret docker-registry my-docker-credentials --docker-username=unclebob --docker-password=secretpwd --docker-email=uncle@bob.org --docker-server=myregistry:5000
```

# API

```bash
kubectl proxy --port=8001&
```
```bash
curl -XDELETE http://localhost/api/v1/pods/nginx # on ressource will delete pods fe 
```

# RBAC

```bash
kubectl config view #describe account logged and different information 
```

# Serviceaccount
```bash
kubectl create sa mysa
kubect set sa deploy nginxapp mysa
```

# Blue Green deploymente (testnet) 
Delete and replace exposure with ";" to execute simultany and have minimum down time

# Canary 
Test update on one pod to test. 
if error => go back
if ok : deploy on all

exemple :
```
                  =>    old |  replicas = go down to 0   |  type=canary  
    SVC 
  Type = canary
                  =>    new |  replicas = go up to       |  type=canary
                            |         old replias number | 
```
``` bash
kubectl scale deploy new-nginx --replicas=3 #to change replicas number
kubectl scale deploy old-nginx --replicas=0 #to change replicas number
```

# Custom Ressources Definition

exemple of backup 
permite to create a perssonal ressources doesnt exist in k8s

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.stable.example.com
spec:
  group: stable.example.com
  versions: 
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              backupType:
                type: string
              image:
                type: string
              replicas:
                type: integer
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    shortNames:
     - bks
    kind: BackUp
```

# Operator

# Troubleshooting

```bash
kubectl get netpol -A # to check network policy existence```
```

Thceck les label selector dans un svc NodePort
** /!\ Si cest  un svc clusterIP on dois se connecter au node minikube ssh **

```bash
kubectl get events #when lost and dont know what to search
kubectl config view # pour voir la config de connexion au node
kubectl auth can-i create pods # pour check les droits 
```

```bash
minikube ssh
sudo -i # go to root
cp /etc/kubernetes/admin.conf /tmp
scp -i $(minikube ssh-key) docker@$(minikube ip):/tmp/admin.conf  ~/.kube/config
```

# Probes

 - readinessProbe
 - livenessProbe
 - startupProbe

 - exec
 - httpGet
 - tcpSocket