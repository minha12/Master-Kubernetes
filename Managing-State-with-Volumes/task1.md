# Kubernetes Volumes

## Key Concepts

1. **Volumes**:
   - Ephemeral: Last the lifetime of a pod.
   - Persistent: Exist beyond the lifetime of a pod and preserve data across restarts.

2. **Volume Types**:
   - **ConfigMap**: Injects configuration data into pods.
   - **DownwardAPI**: Exposes metadata about the pod and node.
   - **emptyDir**: A temporary directory created for a pod that exists for the pod's lifetime.
   - **hostPath**: Mounts a file or directory from the host node's filesystem into the pod.
   - **NFS**: Mounts an existing NFS share into a pod.
   - **PersistentVolumeClaim (PVC)**: Claims persistent storage for a pod.
   - **Secret**: Passes sensitive information to pods.
   - **CSI**: Container Storage Interface for using third-party storage drivers.

3. **Mount Propagation**:
   - **None**: Mounts are isolated.
   - **HostToContainer**: Host mounts are visible to the container.
   - **Bidirectional**: Mounts are shared between the host and container.

4. **Deprecated and Removed Volumes**:
   - Several volume types like `awsElasticBlockStore`, `azureDisk`, `azureFile`, `cephfs`, `cinder`, `gcePersistentDisk`, `glusterfs`, `gitRepo`, `portworxVolume`, `rbd`, and `vsphereVolume` have been deprecated or removed in favor of their corresponding CSI drivers.

## Commands and Usage

1. **ConfigMap**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: configmap-pod
   spec:
     containers:
       - name: test
         image: busybox:1.28
         command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
         volumeMounts:
           - name: config-vol
             mountPath: /etc/config
     volumes:
       - name: config-vol
         configMap:
           name: log-config
           items:
             - key: log_level
               path: log_level
   ```

2. **emptyDir**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pd
   spec:
     containers:
     - image: registry.k8s.io/test-webserver
       name: test-container
       volumeMounts:
       - mountPath: /cache
         name: cache-volume
     volumes:
     - name: cache-volume
       emptyDir:
         sizeLimit: 500Mi
   ```

3. **hostPath**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: hostpath-example-linux
   spec:
     os: { name: linux }
     nodeSelector:
       kubernetes.io/os: linux
     containers:
     - name: example-container
       image: registry.k8s.io/test-webserver
       volumeMounts:
       - mountPath: /foo
         name: example-volume
         readOnly: true
     volumes:
     - name: example-volume
       hostPath:
         path: /data/foo
         type: Directory
   ```

4. **NFS**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pd
   spec:
     containers:
     - image: registry.k8s.io/test-webserver
       name: test-container
       volumeMounts:
       - mountPath: /my-nfs-data
         name: test-volume
     volumes:
     - name: test-volume
       nfs:
         server: my-nfs-server.example.com
         path: /my-nfs-volume
         readOnly: true
   ```

5. **PersistentVolumeClaim**:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: my-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

6. **Secret**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: secret-pod
   spec:
     containers:
       - name: test
         image: busybox:1.28
         command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
         volumeMounts:
           - name: secret-vol
             mountPath: /etc/secret
     volumes:
       - name: secret-vol
         secret:
           secretName: my-secret
   ```

7. **CSI**:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: csi-pv
   spec:
     capacity:
       storage: 5Gi
     volumeMode: Filesystem
     accessModes:
       - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: csi-sc
     csi:
       driver: csi-driver.example.com
       volumeHandle: unique-volume-handle
       fsType: ext4
   ```