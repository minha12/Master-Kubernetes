# What is Pod?

**Definition and Concept:**
- A **Pod** is the smallest and simplest Kubernetes object used for running containers.
- Typically encapsulates one or more containers that share storage, network, and specifications on how to run the containers.
- Often described as a model of an application-specific "logical host" as it simulates a physical or virtual machine in a cloud context.

**Types of Pods:**
1. **Single-container Pods:** Most common use; each Pod wraps a single container. Kubernetes directly manages these Pods rather than the containers themselves.
2. **Multi-container Pods:** These Pods contain tightly coupled containers that need to share resources like storage and networking.

**Usage of Pods:**
- Pods are generally created and managed not directly but through higher-level Kubernetes workload resources such as Deployments, StatefulSets, and DaemonSets.
- These resources manage the lifecycle of Pods, handling tasks like deployment, scaling, and self-healing.

**Pod Management:**
- **Workload Resources:** Use Kubernetes constructs like Deployments for managing stateless Pods and StatefulSets for stateful Pods.
- **Controllers and Templates:** Controllers like those in Deployments use Pod templates defined in specs to create Pods. Changing a Pod’s template leads to the creation of new Pods but does not affect existing ones.
- **Pod Scaling:** Individual Pods shouldn't handle replication or scaling; these are managed by controllers that handle multiple Pods.

**Advanced Pod Concepts:**
- **Init Containers:** Special containers that run before application containers for setup scripts or other initialization tasks.
- **Ephemeral Containers:** Can be added temporarily to running Pods for troubleshooting or debugging during their lifecycle.
- **OS Specifications:** Pods can specify OS requirements using `.spec.os.name`, essential for clusters running multiple OS types.
- **Static Pods:** Managed directly by the kubelet daemon on a node without API server monitoring, typically used for running control plane components.

**Networking and Communication:**
- **Unique IP Assignment:** Each Pod gets a unique IP address, allowing direct communication between Pods across the cluster network.
- **Shared Resources:** Containers in the same Pod share network resources (like IP addresses) and can communicate using standard inter-process communications like POSIX shared memory.
  
**Security and Observability:**
- **SecurityContext:** Defines security settings at the Pod level, such as permissions and user/group IDs that containers must run under.
- **Probes:** Liveness, readiness, and startup probes are used to check the health of containers within Pods, improving reliability through automatic handling of unhealthy containers.

**Work with Pods:**
- **Creating and Managing Pods:** Typically through YAML files applied with `kubectl apply -f`, though direct Pod management is less common outside of specific cases like troubleshooting.
- **Pod Lifecycles and Controllers:** Understand the lifecycle of Pods and how controllers manage their creation, destruction, and updating in response to template changes or failures.

**Best Practices:**
- Avoid directly interacting with individual Pods for application management; instead, use higher-level resources that appropriately handle scaling, fault tolerance, and updates.
- Utilize init and sidecar containers for specialized tasks that are peripheral but necessary to the main application containers running in the Pod.

# Pods Resource Management and QoS Configuration

1. **Setup**: Create a `my-app-deployment.yaml` file for configuring a Kubernetes deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app-container
        image: nginx:latest
        resources:
          requests:
            memory: "256Mi"  # Minimum memory needed to run efficiently
            cpu: "500m"      # 0.5 CPU units (500 milliCPU), minimum needed
          limits:
            memory: "512Mi"  # Maximum memory this container is allowed to use
            cpu: "1"         # Maximum of 1 CPU unit (1000 milliCPU) allowed
```

### Apply the Deployment

- **Deploy** the application using `kubectl` with the following command:

```bash
kubectl apply -f my-app-deployment.yaml
```

### Verify the Deployment and Check QoS Class

- **Check Pod Status**:

```bash
kubectl get pods
```

- **Check QoS Class of the Pod**:

```bash
kubectl get pod -o=custom-columns=NAME:.metadata.name,QOS:.status.qosClass
```

### What This Configuration Demonstrates:

- **Resource Requests and Limits**: You have specified both requests and limits for memory and CPU:
  - **Requests**: Are used by the Kubernetes scheduler to determine which node to place the Pod on based on availability. Ensuring that there are at least these resources available prevents the Pod from failing due to insufficient resources.
  - **Limits**: Act as a hard cap on the resources a container can use and are enforced by the kubelet. These prevent the container from using excess resources that could destabilize other components on the same node.

- **Quality of Service (QoS) Class**:
  - **Burstable**: Since the request and limit values are not equal for both CPU and memory, the Pod falls into the `Burstable` QoS class. This class is given to Pods that have defined resource requests lower than the limits. It ensures that the Pod can exceed its resource requests if additional resources are available on the node but can be throttled back if necessary.

### Why This Matters:

- **Efficient Resource Utilization**: Properly setting requests and limits ensures that your resources are allocated efficiently across your cluster. This avoids resource starvation or bottlenecks.
  
- **Predictable Application Performance**: By understanding and configuring the QoS classes, you ensure that your applications have predictable performance patterns based on resource availability and constraints.

## Assign CPU Resources to Containers and Pods

Here's a summary of all the commands used in the context of managing CPU requests and limits for containers in Kubernetes, along with their purposes:

1. **Check kubectl version:**
   ```bash
   kubectl version
   ```
   This command displays the current version of the kubectl tool and the Kubernetes cluster it is configured to interact with.

2. **Enable metrics-server in Minikube (if using Minikube):**
   ```bash
   minikube addons enable metrics-server
   ```
   This command enables the metrics-server addon in Minikube, which is necessary for gathering resource usage metrics such as CPU and memory.

3. **Check for the availability of the resource metrics API:**
   ```bash
   kubectl get apiservices
   ```
   This command lists available API services, and you can check if `metrics.k8s.io` is listed among them, indicating that the metrics API is running.

4. **Create a namespace:**
   ```bash
   kubectl create namespace cpu-example
   ```
   This command creates a new namespace called `cpu-example`, which helps in isolating resources used in this tutorial from other parts of the cluster.

5. **Create a Pod with specific CPU requests and limits:**
   ```bash
   kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit.yaml --namespace=cpu-example
   ```
   This command creates a Pod according to a configuration file found at the specified URL, within the `cpu-example` namespace. The configuration includes CPU requests and limits.

6. **Get Pod information:**
   ```bash
   kubectl get pod cpu-demo --namespace=cpu-example
   ```
   This command retrieves the basic information about the `cpu-demo` Pod within the `cpu-example` namespace.

7. **View detailed Pod information:**
   ```bash
   kubectl get pod cpu-demo --output=yaml --namespace=cpu-example
   ```
   This command shows detailed information about the `cpu-demo` Pod, including its CPU requests and limits, in YAML format.

8. **Fetch the metrics for the Pod:**
   ```bash
   kubectl top pod cpu-demo --namespace=cpu-example
   ```
   This command fetches and displays CPU and memory usage statistics for the `cpu-demo` Pod.

9. **Create a Pod with a CPU request larger than the capacity of any Node:**
   ```bash
   kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example
   ```
   This command attempts to create another Pod that requests more CPU resources than are available on any single Node, to demonstrate scheduling failures due to insufficient resources.

10. **Describe the Pod to view scheduling problems:**
    ```bash
    kubectl describe pod cpu-demo-2 --namespace=cpu-example
    ```
    This command displays detailed information about the `cpu-demo-2` Pod, including events and status messages indicating why the Pod cannot be scheduled (e.g., insufficient CPU resources).

11. **Delete a Pod:**
    ```bash
    kubectl delete pod cpu-demo --namespace=cpu-example
    kubectl delete pod cpu-demo-2 --namespace=cpu-example
    ```
    These commands delete the specified Pods. You'll need to replace `cpu-demo` with the name of the Pod you want to delete, as shown in two examples.

12. **Delete the namespace:**
    ```bash
    kubectl delete namespace cpu-example
    ```
    This command deletes the `cpu-example` namespace, cleaning up all resources associated with it.


## Assigning Memory Resources

Assigning memory resources to containers and Pods in Kubernetes involves specifying memory requests and limits. This ensures that containers have sufficient memory and do not exceed their defined resource allocation, creating a balanced and efficient cluster environment.

**Before You Begin:**

1. **Ensure Adequate Memory on Nodes**: At least 300 MiB of memory should be available on each node.

2. **Install Metrics Server** (if needed, particularly for Minikube):
   ```bash
   minikube addons enable metrics-server
   ```

3. **Verify Metrics Server Deployment**:
   ```bash
   kubectl get apiservices
   ```

**Create a Namespace**:
This isolates resources created for this exercise from other cluster resources.
```bash
kubectl create namespace mem-example
```

**Specify Memory Request and Limit**:
1. **Define a Pod with Memory Specifications**:
   Here’s how you can specify memory requests and limits in a Pod manifest:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: memory-demo
     namespace: mem-example
   spec:
     containers:
     - name: memory-demo-ctr
       image: polinux/stress
       resources:
         requests:
           memory: "100Mi"
         limits:
           memory: "200Mi"
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
   ```

   This configuration requests 100 MiB of memory and sets a limit of 200 MiB on memory usage.

2. **Create the Pod**:
   ```bash
   kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit.yaml --namespace=mem-example
   ```

3. **Verify and Observe the Pod**:
   Check if the Pod is running and inspect its memory usage:
   ```bash
   kubectl get pod memory-demo --namespace=mem-example
   kubectl top pod memory-demo --namespace=mem-example
   ```

**Exceed a Container's Memory Limit**:
1. **Create Another Pod That Exceeds Memory Limits**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: memory-demo-2
     namespace: mem-example
   spec:
     containers:
     - name: memory-demo-2-ctr
       image: polinux/stress
       resources:
         requests:
           memory: "50Mi"
         limits:
           memory: "100Mi"
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
   ```

   This Pod tries to use more memory than its limit by attempting to use 250 MiB.

2. **Create and Monitor the Pod**:
   ```bash
   kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-2.yaml --namespace=mem-example
   kubectl describe pod memory-demo-2 --namespace=mem-example
   ```

   Observe the behavior and potential OOM (Out-Of-Memory) kills due to memory limit exceedance.

**Specify an Excessive Memory Request**:
1. **Attempt to Deploy a Pod with Unreasonably High Memory Request**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: memory-demo-3
     namespace: mem-example
   spec:
     containers:
     - name: memory-demo-3-ctr
       image: polinux/stress
       resources:
         requests:
           memory: "1000Gi"
         limits:
           memory: "1000Gi"
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
   ```

2. **Create the Pod and Observe**:
   ```bash
   kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-3.yaml --namespace=mem-example
   kubectl get pod memory-demo-3 --namespace=mem-example
   kubectl describe pod memory-demo-3 --namespace=mem-example
   ```

   This Pod will remain in the Pending state due to the excessive memory request.

**Clean Up**:
Delete the namespace to clean up all associated resources:
```bash
kubectl delete namespace mem-example
```

## Schedule GPUs

To schedule GPUs in a Kubernetes cluster, you'll need to configure your cluster to recognize and handle GPU resources using device plugins. This setup involves ensuring GPU drivers and specific Kubernetes plugins are properly installed and utilized. Below is a structured guide detailing the steps and commands needed for GPU scheduling:

---

### Before You Begin
- **Install GPU Drivers**: Depending on the GPU vendor (AMD, NVIDIA, Intel), install the necessary GPU drivers on the nodes that will handle GPU tasks.

- **Install Device Plugins**: Follow the vendor-specific instructions to install Kubernetes device plugins that allow the cluster to utilize the GPUs.
    - AMD: [Vendor instructions]
    - NVIDIA: [Vendor instructions]
    - Intel: [Vendor instructions]

### Expose GPU Resources
- Once device plugins are installed, they will advertise custom resources like `amd.com/gpu` or `nvidia.com/gpu`.

### Request GPUs in Pod Specifications
- **Create a Pod that requests GPU resources**. Here's an example manifest for a pod requesting one GPU:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: example-vector-add
   spec:
     restartPolicy: OnFailure
     containers:
       - name: example-vector-add
         image: "registry.example/example-vector-add:v42"
         resources:
           limits:
             gpu-vendor.example/example-gpu: 1  # Requesting 1 GPU
   ```
   Apply this configuration:
   ```bash
   kubectl apply -f example-vector-add.yaml
   ```

### Manage Clusters with Different Types of GPUs
- **Labeling Nodes**: Label your nodes based on the type of GPU they contain:
  ```bash
  kubectl label nodes node1 accelerator=example-gpu-x100
  kubectl label nodes node2 accelerator=other-gpu-k915
  ```
- This kind of labeling helps during pod scheduling to match pods to appropriate nodes based on GPU types.

### Automatic Node Labeling with NFD
- **Deploy Node Feature Discovery (NFD)** to automatically detect and label hardware features including GPUs on nodes:
    - NFD will generate labels such as `feature.node.kubernetes.io/gpu-vendor.example/installed-memory`.
    - It can also be configured to add annotations, extended resources, and node taints.

### Advanced Scheduling Using Affinity
- **Use node affinity** to schedule pods on nodes with specific GPU types and features:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: example-vector-add
   spec:
     restartPolicy: OnFailure
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: "gpu.gpu-vendor.example/installed-memory"
                   operator: Gt
                   values: ["40535"]
                 - key: "feature.node.kubernetes.io/pci-10.present"
                   values: ["true"]
     containers:
       - name: example-vector-add
         image: "registry.example/example-vector-add:v42"
         resources:
           limits:
             gpu-vendor.example/example-gpu: 1
   ```
   This configuration ensures that the pod `example-vector-add` is scheduled on a node with the required GPU resources and features.


## Pulling an Image from a Private Registry 

Pulling an image from a private registry in Kubernetes involves several steps, including creating a secret to store your registry credentials and referencing that secret in your Pod configuration. Here's how you can do it, explained with command usages and descriptions:

---

**Prerequisites Check:**

- Log in to Docker Hub or your private registry using the Docker CLI to authenticate and generate a configuration file (`config.json`):
  ```bash
  docker login
  ```

**Inspect Docker Configuration:**
- View your Docker `config.json` to verify it contains your authentication token:
  ```bash
  cat ~/.docker/config.json
  ```

**Create a Kubernetes Secret for Registry Credentials:**
- Use your existing Docker credentials to create a Kubernetes secret:
  ```bash
  kubectl create secret generic regcred \
      --from-file=.dockerconfigjson=$HOME/.docker/config.json \
      --type=kubernetes.io/dockerconfigjson
  ```

**Alternatively, Create a Secret via Command Line (not recommended due to security reasons):**
- Manual creation of a docker-registry type secret:
  ```bash
  kubectl create secret docker-registry regcred \
      --docker-server=<your-registry-server> \
      --docker-username=<your-name> \
      --docker-password=<your-pword> \
      --docker-email=<your-email>
  ```
  Replace `<your-registry-server>`, `<your-name>`, `<your-pword>`, and `<your-email>` with your registry's details.

**Inspect the Created Secret:**
- View the created secret in YAML format:
  ```bash
  kubectl get secret regcred --output=yaml
  ```

- Decode the `.dockerconfigjson` from the secret to understand its contents:
  ```bash
  kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
  ```

**Creating a Pod that Uses the Secret:**
- Example Pod definition to use the secret for pulling an image from a private registry:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: private-reg
  spec:
    containers:
    - name: private-reg-container
      image: <your-private-image>
    imagePullSecrets:
    - name: regcred
  ```
  Replace `<your-private-image>` with your private image URL.

- Save the above Pod definition into a file named `my-private-reg-pod.yaml`, manually or using:
  ```bash
  curl -L -o my-private-reg-pod.yaml https://k8s.io/examples/pods/private-reg-pod.yaml
  ```

- Apply the Pod definition:
  ```bash
  kubectl apply -f my-private-reg-pod.yaml
  ```

- Verify the Pod is running:
  ```bash
  kubectl get pod private-reg
  ```

**Troubleshoot Pod Image Pull Issues:**
- If the Pod fails to start, inspect it for errors:
  ```bash
  kubectl describe pod private-reg
  ```
  Look for events related to failed image pulls or secret retrievals.

---

Following these steps helps ensure that your Kubernetes Pods can securely access and pull images from private registries using credentials stored as Kubernetes Secrets.