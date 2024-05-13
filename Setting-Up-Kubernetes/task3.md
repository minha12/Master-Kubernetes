# Example: Deploying an HTTP Echo Server

We will use the k8s.gcr.io/echoserver Docker image, which is a simple HTTP server that responds with the headers it received. This is an excellent way for beginners to understand how to interact with applications running within a Kubernetes cluster.

Step 1: Create a Deployment
Deploy the echo server using the following kubectl command:

`kubectl create deployment echo-server --image=k8s.gcr.io/echoserver:1.10`

This command creates a deployment named echo-server using the echoserver image.

Step 2: Expose the Deployment as a Service
To access the echo server from outside the Kubernetes cluster, expose it as a service:

`kubectl expose deployment echo-server --type=NodePort --port=8080`

This command creates a service that exposes the echo server on a port that will be dynamically assigned by Minikube.

Step 3: Find the Service URL
Minikube provides an easy command to find the URL of the exposed service:

`minikube service echo-server --url`

This command will output a URL that you can use to access the echo server from your web browser or using a tool like curl.

Step 4: Access the Service
Open a web browser and navigate to the URL provided by the previous command, or use curl to access the service:

`curl $(minikube service echo-server --url)`

This will send a request to your echo server, and you should see a response that displays the headers sent by your HTTP client (in this case, curl).

Step 5: Monitor the Deployment
You can check the status of your deployment using:

`kubectl get deployment echo-server`

And for the service:

`kubectl get service echo-server`

You can also see the details and events related to the deployment:

`kubectl describe deployment echo-server`

And for the service:

`kubectl describe service echo-server`

Step 6: Update the Deployment (Optional)
If you need to update the image or change the number of replicas, you can do so using the kubectl set or kubectl scale commands. Here’s how to scale the deployment to run 3 instances of the echo server:

`kubectl scale deployment echo-server --replicas=3`

After a few moments, you can check the status and see that you now have three replicas:

`kubectl get pods`

Step 7: Clean Up
Once you are done experimenting, you can delete the deployment and service to clean up:

`kubectl delete service echo-server`
`kubectl delete deployment echo-server`

This ensures no resources are left running that could incur costs or utilize system resources.
