# Using Persistent Storage

In Kubernetes, managing storage involves defining Persistent Volumes (PV) and Persistent Volume Claims (PVC) to provide applications with a reliable and potentially stateful storage mechanism. Below is an example of how to configure and use persistent storage in a Kubernetes environment. We will proceed through the tasks you outlined.

## Task 1: Create a Persistent Volume using a YAML file

Let's define a simple Persistent Volume using a YAML configuration.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```
Here, we use hostPath for simplicity, which means this will only work on a single node cluster like Minikube. For multi-node clusters, you'd use other options such as nfs or a cloud provider specific storage like AWS's EBS, Google's Persistent Disk, or Azure's Disk Storage.

Apply this configuration with the following command: `kubectl apply -f persistent-volume.yaml`

## Task 2: Create a Persistent Volume Claim and bind it to your PV

Now, create a PVC to claim storage from the PV defined earlier.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeName: example-pv
```

This claim will bind to the Persistent Volume example-pv due to the explicit volumeName and matching accessModes and storage capacity. 

Apply this configuration with the following command: `kubectl apply -f persistent-volume-claim.yaml`

## Task 3: Deploy an application that uses this PVC for storage

Next, let's deploy a simple application that uses this PVC. Here, we’ll deploy an Nginx server that stores its logs on our persistent storage.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: log-volume
          mountPath: /var/log/nginx
      volumes:
      - name: log-volume
        persistentVolumeClaim:
          claimName: example-pvc
```

Apply this deployment with the following command: `kubectl apply -f app-using-pvc.yaml`

## Task 4: Test data persistence by deleting and redeploy

To test the persistence of the storage, perform the following:

Check the existing logs or create a file in `/var/log/nginx` within the Nginx pod.
Delete the deployment with the following command: `kubectl delete deployment nginx-deployment`
Redeploy the application with the following command: `kubectl apply -f app-using-pvc.yaml`
Check the logs or the file within `/var/log/nginx`. The data should still be there, indicating that your PV and PVC kept the data through the deployment lifecycle.

