# Run Single-Instance Stateful Application

This page shows you how to run a single-instance stateful application in Kubernetes using a PersistentVolume and a Deployment
## Deploy Mariadb
You can run a stateful application by creating a Kubernetes Deployment and connecting it to an existing PersistentVolume using a PersistentVolumeClaim. 

#### Deploy the Mariadb Secret of the YAML file
to create maridb secret you must definition with new values :

$echo -n "secret" | base64

```
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-single-secret
type: Opaque
data:
  password: c2VjcmV0

```
kubectl create -f ../mariadb-secret.yaml
#### Deploy the Mariadb Configmap of the YAML file
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mariadb-configmap
data:
  database_url: mariadb-instance-service
```

kubectl create -f ../mariadb-configmap.yaml
#### Deploy the Mariadb Volume of the YAML file
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-single-instance-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/mariadb/single-instance"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-single-instance-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```
kubectl create -f ../mariadb-volume.yaml


#### Deploy the Mariadb Deployment of the YAML file
```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-instance-service
spec:
  selector:
    app: mariadb
  type: NodePort
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30031
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - image: mariadb:10.4
        name: mariadb
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-single-secret
              key: password
        ports:
        - containerPort: 3306
          name: maridb
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-persistent-storage
        persistentVolumeClaim:
          claimName: mariadb-single-instance-pv-claim     
```
kubectl create -f ../mariadb-deployment.yaml
