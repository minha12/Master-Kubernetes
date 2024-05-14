# Setting Up Ingress Controllers

## Objective:
Implement an Ingress Controller to manage access to services in a cluster.

## Tasks:

- Install an Ingress Controller like Nginx or Traefik using Helm charts or YAML files.
- Configure Ingress rules to route traffic to different services based on the request URL.
- Test the configuration by accessing your services through the Ingress endpoint.

> Setting up an Ingress Controller in a Kubernetes cluster involves several steps, from installing the necessary software components to defining rules to direct traffic to your services. Below, I will illustrate how to deploy Nginx as an Ingress Controller using a Helm chart, then set up basic Ingress rules, and briefly describe how to test these configurations.

## Step 1: Install an Ingress Controller (NGINX)

First, ensure helm is installed. Helm simplifies the deployment of complex applications on Kubernetes. 

```
brew install helm
```

Add the Helm repository for the NGINX Ingress Controller:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
Install the NGINX Ingress Controller using Helm:

```
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-basic --create-namespace --set controller.publishService.enabled=true
```

- `helm install nginx-ingress`: This part of the command tells Helm to install a new release and names that release nginx-ingress. Helm uses releases to track installations of applications.

- `ingress-nginx/ingress-nginx`: This specifies the chart to be installed. The first ingress-nginx refers to the Helm repository name, and the second ingress-nginx is the chart name within that repository.

- `--namespace ingress-basic`: Specifies the Kubernetes namespace to install the Ingress Controller into. If this namespace does not exist, it will be created due to the next flag.

- `--create-namespace`: This triggers Helm to create the ingress-basic namespace if it doesn't already exist.

- `--set controller.publishService.enabled=true`: This configuration parameter sets the publishService field under controller in the chart's values to true. Typically, this tells the NGINX Ingress controller to create a Kubernetes service resource that exposes the NGINX controller to the internet, often through a LoadBalancer service.

This command installs the NGINX Ingress Controller in the ingress-basic namespace and configures it to be exposed externally.

## Step 2: Configure Ingress Rules

Create a YAML file to define Ingress rules. This example assumes you have two services, service-one and service-two, that you want to expose:

`ingress-rules.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: ingress-basic
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /service-one(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: service-one
            port:
              number: 80
      - path: /service-two(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: service-two
            port:
              number: 80
```

### Top-Level Fields
- `apiVersion: networking.k8s.io/v1`: Specifies the API version for this resource. For Ingress, `networking.k8s.io/v1` is the stable version and is widely supported.

- `kind: Ingress`: Indicates that the resource type is an Ingress. Ingress resources define rules for inbound traffic to reach cluster services.

- `metadata`: Provides metadata about the Ingress resource, such as its name and namespace.

- `name: example-ingress`: Defines the name of the Ingress resource. Names must be unique within a namespace.
- `namespace`: ingress-basic: Specifies the Kubernetes namespace in which this Ingress resource is created. Namespaces are a way to divide cluster resources between multiple users.

### Spec Section
The spec section is where the behavior of the Ingress is defined:

- `ingressClassName: nginx`: Specifies the Ingress Controller that should implement this Ingress resource. Ingress Controllers are typically deployed separately and are responsible for fulfilling the traffic routing described by Ingress resources. Here, it indicates that this Ingress should be handled by an Ingress Controller named "nginx", which refers to the NGINX Ingress Controller.
- Rules Section
+ `rules`: A list of rules applied to incoming traffic where each rule allows for defining different hosts and paths that the Ingress will route.
+ `http`: Indicates that these rules apply to HTTP traffic.
+ `paths`: A list of path elements, each consisting of a path and a backend definition. Each path associated with a backend defines how traffic should be redirected when the path matches the incoming request URL.
+ Path Definitions
For each item in paths:
  + `path: /service-one(/|$)(.*) and path: /service-two(/|$)(.*)` These strings define the URL paths that will trigger the rules. The regex `/service-one(/|$)(.*)` means "match any URL that starts with /service-one, optionally followed by a /, and then any sequence of characters." This allows flexible routing, for example:
    `/service-one`
    `/service-one/`
    `/service-one/more/path`
  The `ImplementationSpecific` path type allows the NGINX Ingress Controller to interpret the regex as per its own implementation standards, which enables regex matching in NGINX.
  + `pathType: ImplementationSpecific` Tells the Ingress controller that the path should be interpreted according to the rules specific to the Ingress controller itself. This type enables using controller-specific features like regex path matching supported by NGINX.

### Backend Definition
For each path rule:

- `backend`: Defines how to dispatch the traffic that matches the corresponding path:
service: Specifies the Kubernetes Service to which traffic should be directed.
name: service-one and name: service-two: Names of the Kubernetes Services that will receive the traffic.
- `port: number: 80`: The port on the Service to which the traffic should be sent. Both services are set to receive traffic on port 80, which is the standard port for HTTP.

This configuration directs requests to `/service-one` to the service-one service and `/service-two` to the service-two service. The rewrite-target annotation is used to strip the path prefix before forwarding the request to the backend services.

Apply the Ingress rules:
```
kubectl apply -f ingress-rules.yaml
```

Minikube includes a command called tunnel which runs in a separate terminal or background process. This command creates a network route on the host to the Minikube VM, allowing Kubernetes services of type LoadBalancer to be accessed via an externally accessible IP:

```
minikube tunnel
```

## Step 3: Test the Configuration

After deploying the Ingress and the services, you'll want to ensure it works as expected:

1. Determine the IP address or DNS name of your Ingress Controller. If you are on a cloud provider, the LoadBalancer service should provide an external IP:

```
kubectl get services -n ingress-basic
```

```
NAME                                               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.101.40.59     127.0.0.1     80:32030/TCP,443:30849/TCP   37m
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.104.159.249   <none>        443/TCP                      37m
```

2. Access your services through a browser or using curl:

Replace INGRESS_IP with the actual IP address or hostname. (It is the EXTERNAL-IP for `nginx-ingress-ingress-nginx-controller` in the case above)

```
curl http://INGRESS_IP/service-one
curl http://INGRESS_IP/service-two
```

These commands should route to service-one and service-two respectively based on the defined Ingress rules.
