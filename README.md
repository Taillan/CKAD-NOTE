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
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    securityContext: #Container Security context
      allowPrivilegeEscalation: false
```
