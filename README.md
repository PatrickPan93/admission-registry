# admission-registry
Adding sidecar containers & volumes in a labeled namespace by admission Controller mutating
Adding annotation to avoid inject sidecar "sidecar-injector-webhook.poon.me/inject: false"
Remember to enable admission plugin in kube-apiserver:
--enable-admission-plugins=NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook

### 1. Using cert-generated.sh to generated certs for application
```shell
# should have a look at this script, modify the param you need
.cert/cert-genrated.sh
```

### 2. Modify the app-configmap ( add sidecar container & volumes you need )
```yaml
# For example 
apiVersion: v1
kind: ConfigMap
metadata:
  name: sidecar-injector-webhook-configmap
data:
  sidecarconfig.yaml: |
    containers:
    - name: nginx
      image: nginx:1.18.0
      imagePullPolicy: IfNotPresent
      ports:
      - containerPort: 80
      resources:
        requests:
          cpu: "50m"
          memory: "50Mi"
        limits:
          cpu: "100m"
          memory: "100Mi"
      volumeMounts:
      - name: nginx-conf
        mountPath: /etc/nginx
    - name: nginx2
      image: nginx:1.18.0
      imagePullPolicy: IfNotPresent
      ports:
      - containerPort: 80
      resources:
        requests:
          cpu: "50m"
          memory: "50Mi"
        limits:
          cpu: "100m"
          memory: "100Mi"
      volumeMounts:
      - name: nginx-conf
        mountPath: /etc/nginx
    volumes:
    - name: nginx-conf
      configMap:
        name: nginx-configmap
```
### 3.Deploy apps to kubernetes
```shell
# modify the image info as yours
kubectl apply -f manifests/deployment.yaml
```

### 4.Apply admission rules after application applied 
```shell
kubectl apply -f manifests/mutatingwebhook.yaml
```
### Notice
```yaml
# using sidecar configmap if needed
kubectl apply -f configmap/sidecar-nginx-configmap.yaml
```