## Configuring Redis using a ConfigMap

This example walks you through the process of configuring a Redis Pod in Kubernetes using a ConfigMap. Below is a detailed explanation of each step and the knowledge required to execute them. This guide assumes you are familiar with basic Kubernetes concepts such as Pods, ConfigMaps, and kubectl commands.

### Objectives

1. **Create a ConfigMap with Redis configuration values**: A ConfigMap allows you to decouple configuration artifacts from image content to keep containerized applications portable.
2. **Create a Redis Pod that uses the ConfigMap**: A Pod is the smallest deployable unit of computing in Kubernetes.
3. **Verify that the configuration was correctly applied**: Ensuring that the Redis server runs with the desired configuration settings.

### Steps Explained

#### Step 1: Create the ConfigMap
You start by creating a ConfigMap named `example-redis-config`. This ConfigMap initially contains an empty Redis configuration. This is done using a YAML file, and this file is applied to your Kubernetes cluster with `kubectl apply`. The purpose here is to prepare a vessel in which Redis specific configurations can be stored.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
```

#### Step 2: Deploy the Redis Pod
With the ConfigMap in place, you then deploy a Redis Pod using another YAML file (obtained from a URL). This manifest describes a Pod that runs a Redis container. Important parts of this configuration include:
```
kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf

```

- **Volumes**: It defines a volume named `config` that is sourced from the previously created ConfigMap (`example-redis-config`). The `configMap` field specifies that the `redis-config` key should be exposed in the Pod as a file named `redis.conf`.

- **Volume Mounts**: It mounts the `config` volume inside the container at the path `/redis-master`. This way, Redis within the container reads its configuration from `/redis-master/redis.conf`.

### Step 3: Examine and Verify the ConfigMap and Redis Configuration
To review the newly created objects and their statuses, use the following commands:

```bash
kubectl get pod/redis configmap/example-redis-config
```

This command fetches the status of the `redis` Pod and the `example-redis-config` ConfigMap. Look for the `Running` status on the Pod which indicates it's up and operational.

Next, check the content of the ConfigMap to confirm it currently holds an empty Redis configuration:

```bash
kubectl describe configmap/example-redis-config
```

This command provides detailed information about the ConfigMap, confirming that `redis-config` is indeed empty as expected.

To further validate the Redis configuration within the Pod, connect to the Redis CLI with:

```bash
kubectl exec -it redis -- redis-cli
```

Once inside the CLI, check the Redis server's configuration:

```bash
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

These commands return the default values (`0` and `noeviction`) for `maxmemory` and `maxmemory-policy`, respectively, since no custom values have been configured yet.

### Step 4: Update the ConfigMap
Update the `example-redis-config` ConfigMap with specific Redis configuration values. First, modify the `example-redis-config.yaml` to include new settings:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
```

Apply the updated ConfigMap:

```bash
kubectl apply -f example-redis-config.yaml
```

Confirm that the ConfigMap was updated successfully:

```bash
kubectl describe configmap/example-redis-config
```

The output should now show the newly added values under the `redis-config` key.

### Step 5: Restart the Pod to Apply Configuration Changes
Since ConfigMaps do not automatically update configurations inside running Pods, you need to recreate the Pod to apply the changes:

```bash
kubectl delete pod redis
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

This sequence of commands deletes the old Pod and re-creates it, allowing it to re-mount the updated ConfigMap with the new configurations.

### Step 6: Confirm Changes
Access the Redis CLI again to verify if the Redis server now uses the updated configurations:

```bash
kubectl exec -it redis -- redis-cli
```

Check the configuration settings once more:

```bash
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

You should now see `2097152` (or approximately `2mb` converted into bytes) for `maxmemory` and `allkeys-lru` for `maxmemory-policy`, reflecting the values set in the updated ConfigMap.

### Step 7: Clean up
After confirming that everything works as expected, clean up the created resources to avoid unnecessary resource usage and costs:

```bash
kubectl delete pod/redis configmap/example-redis-config
```

These commands will delete the Redis Pod and the ConfigMap from your Kubernetes cluster.