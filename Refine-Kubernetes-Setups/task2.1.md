# ConfigMaps

ConfigMaps in Kubernetes serve as a way to externalize and manage the configuration data you would typically place in your application's configuration files. They enable you to keep containerized applications portable and able to be configured within different environments without rebuilding your application images. This is particularly useful in various stages of software development such as testing, staging, and production.

Here is a step-by-step explanation of how you can create, configure, and use ConfigMaps in a Kubernetes environment.

## Step 1: Creating a ConfigMap
From Files
To create a ConfigMap from files or directories, use the kubectl create configmap command. For instance:

Create a directory and navigate into it:
```
mkdir -p configure-pod-container/configmap/
cd configure-pod-container/configmap/
```

Download sample configuration files into this directory:
```
wget https://kubernetes.io/examples/configmap/game.properties -O game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O ui.properties
```

Create a ConfigMap named game-config from files in the directory:

```
kubectl create configmap game-config --from-file=.
```

From Literal Values
To create a ConfigMap directly from literal values:
```
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
```

## Step 2: Viewing ConfigMaps
To check the details of the created ConfigMap, use:

```
kubectl describe configmaps game-config
```
```
kubectl get configmaps special-config -o yaml
```

## Step 3: Using ConfigMaps in Pods
Environment Variables
You can reference ConfigMap values in a Pod as environment variables. Here’s an example of defining a Pod that uses ConfigMap data as an environment variable:
```yaml
piVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: registry.k8s.io/busybox
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.how
```
Save this as pod.yaml and create the Pod:
```
kubectl apply -f pod.yaml
```

Volume Mounts
ConfigMaps can also be mounted as volumes. Each key becomes a file in the volume. Here’s an example Pod configuration that mounts a ConfigMap named special-config into a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: registry.k8s.io/busybox
      command: ["/bin/sh", "-c", "cat /etc/config/SPECIAL_TYPE"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
```
Create this Pod to see your ConfigMap in action:
```
kubectl apply -f pod-with-volume.yaml
```
Q: what is Kubernetes' volumes?

A: In the context of Kubernetes, "volumes" refers to a directory, possibly with data in it, which is accessible to the containers in a pod. A volume can be used for multiple purposes, in one or many pods, and is meant to exist as long as the pod to which it is attached exists. Thus, volumes outlive any containers that run within the pod, and data is preserved across container restarts.

- Key Characteristics of Kubernetes Volumes:
Persistence: A Kubernetes volume's lifecycle is tied to the pod that encloses it. Even if the pod moves between nodes, the volume persists throughout the life of the pod. If a pod is deleted, the data in the volume can be lost unless the volume type inherently supports data persistence.
    + Sharing: Volumes can be shared between containers in the same pod. Each container within the pod can mount the same volume to access shared data which facilitates data exchange and persistence in multi-container pods.

    + Variety of Sources: Kubernetes supports several types of volumes. Each type of volume is backed by a different underlying storage mechanism which could be local disk storage on the node, a network storage like NFS, or a cloud provider-specific storage service, among others.

- Types of Kubernetes Volumes:
    + emptyDir: A simple empty directory used for storing transient data. It is initially empty and all containers in the pod can read from and write to the files in the emptyDir volume.

    + hostPath: A volume that mounts a file or directory from the host node’s filesystem into your pod.

    + persistentVolumeClaim (PVC): An abstract storage unit provided by a PersistentVolume which has a lifecycle independent of any individual pod that uses the PVC.

    + configMap and secret: Special types of volumes that allow you to pass configuration data or secrets into your pods. The ConfigMap or secret must be created before it is mounted as a volume.

    + nfs: An NFS (Network File System) volume that mounts an existing NFS share into your pod.

    + cloud provider-specific storage: Volumes that use the underlying cloud provider’s storage system, such as AWS EBS, Azure Disk, or Google Persistent Disk.

- Usage Examples:
    + Data Persistence: Volumes like PVCs allow for data persistence across pod restarts and even pod re-scheduling across different nodes.

    + Sharing Data: Using an emptyDir in a multi-container pod, where one container might generate or download data to be used by another container in the same pod.

    + Configuration and Secrets: configMap and secret volumes help manage application configurations and sensitive data, allowing for dynamic changes and secure storage of credentials.

Volumes in Kubernetes provide an essential mechanism for data durability and sharing in containerized environments, bridging the gap between stateless applications and stateful application needs.

Q: What is the difference when ConfigMaps mounted as volumes?

A: When ConfigMaps are mounted as volumes in Kubernetes, they provide a flexible way to inject configuration data into containers at runtime. Here’s how ConfigMaps mounted as volumes differ from other methods of using ConfigMaps, such as through environment variables:

1. File-based Access:

    - Volumes: When a ConfigMap is mounted as a volume, each key in the ConfigMap becomes an individual file on the file system of the container. The value of each key is the content of the corresponding file. This allows applications that read configuration data from files at a specific path to be easily configured.
    - Environment Variables: Conversely, when using ConfigMaps as environment variables, the values are set as environment variables in the container’s environment. This is beneficial for applications designed to consume configuration from environment variables.
2. Dynamic Updates:

    - Volumes: Kubernetes has the capability to update the files in the volume dynamically as the ConfigMap is updated (if the mounted ConfigMaps are references and not copied). This keeps the application’s configuration up to date without restarting the container. However, note that not all applications can dynamically re-read their configuration files without a restart.
    - Environment Variables: Environment variables do not dynamically update in response to changes in the ConfigMap while the container is running. To reflect changes, the pod usually needs to be recreated.
3. Filesystem Integration:

    - Volumes: Mounting as a volume integrates smoothly with applications that require their configuration files to be present on the file system, potentially under a specific directory structure. This mimics a more traditional file-based configuration system which some legacy systems or specific frameworks might require.
    - Environment Variables: Using environment variables is generally straightforward for modern, cloud-native applications designed to consume values directly from the environment, but it doesn't suit applications that strictly need file-based configurations.
4. Granularity and Structure:

    - Volumes: Since each key in a ConfigMap becomes a separate file, this method can support configurations that need to be expressed as a set of files in a directory. It provides a greater degree of structure, where different configuration aspects are separated across multiple files.
    - Environment Variables: Injecting ConfigMaps as environment variables can be simpler and more direct for passing configuration values but lacks the hierarchical file structure that some applications might need.

## Step 4: Cleanup
After experimenting, clean up your environment to prevent clutter and unnecessary resource consumption:

```
kubectl delete pod example-pod
kubectl delete configmap game-config special-config
```