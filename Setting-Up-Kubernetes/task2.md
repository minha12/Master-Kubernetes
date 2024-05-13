# Kubectl Command Guide

Kubectl is the Kubernetes command-line tool that allows you to run commands against Kubernetes clusters. It is used for deploying applications, inspecting and managing cluster resources, and viewing logs.

1. Checking kubectl Configuration and Connection:
   - View the version of Kubernetes:
     `kubectl version`

   - Check which context you are currently using (which cluster you're interacting with):
     `kubectl config current-context`

   - List all contexts:
     `kubectl config get-contexts`

2. Working with Kubernetes Objects:
   - Get a list of all pods in the current namespace:
     `kubectl get pods`

   - Get a list of all pods across all namespaces:
     `kubectl get pods --all-namespaces`

   - Get a list of resources with detailed information:
     `kubectl get pods -o wide`

   - Create a resource from a YAML file:
     `kubectl apply -f pod.yaml`

   - Delete a resource:
     `kubectl delete -f pod.yaml`

Example of `*.yaml` file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

3. Managing Deployments:
   - Create a deployment:
     `kubectl create deployment nginx --image=nginx`

   - Scale a deployment to 3 replicas:
     `kubectl scale deployment nginx --replicas=3`

   - Update the version of the deployed application:
     `kubectl set image deployment/nginx nginx=nginx:1.19.1`

   - View the rollout status of a deployment:
     `kubectl rollout status deployment/nginx`

   - Rollback to a previous deployment:
     `kubectl rollout undo deployment/nginx`

4. Debugging and Logs:
   - Get logs for a specific pod:
     `kubectl logs my-pod`

   - Attach to a running container:
     `kubectl attach my-pod -i`
     (Troubleshoot: if the command doesn't work, try this `kubectl exec -it my-pod -- /bin/bash`)

   - Execute a command in a container:
     `kubectl exec my-pod -- ls /`

5. Node and Cluster Management:
   - Get a list of nodes:
     `kubectl get nodes`

   - Get details about a node:
     `kubectl describe node my-node`

6. Use of Labels and Selectors:
   - Get pods with specific labels:
     `kubectl get pods -l app=nginx`

   - Label a pod:
     `kubectl label pods my-pod new-label=awesome`

   - Filter the labeled pod:
     `kubectl get pods -l new-label=awesome`

7. Accessing Services:
   - Expose a deployment as a service:
     `kubectl expose deployment nginx --port=80 --type=NodePort`

   - Get details of a service:
     `kubectl describe services nginx`

These examples provide a glimpse of what you can accomplish using kubectl. For more comprehensive usage, including creating and managing more complex Kubernetes resources, refer to the official Kubernetes documentation. Start by experimenting with these commands, which cover a wide variety of basic tasks in Kubernetes management.
