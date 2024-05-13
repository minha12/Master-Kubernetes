# Minikube and Kubernetes Setup on macOS (Apple Silicon)

This guide provides step-by-step instructions to set up Minikube and Kubernetes on a Mac with an M chip.

## Prerequisites

### Install Homebrew
Homebrew is a package manager for macOS. Open a terminal and install Homebrew by executing the following command:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Follow the instructions in the terminal to complete the installation.
> Troubleshoot: Hombrew will be installed in `/opt/homebrew/bin`, if it is not added to path:
> (1) Run this command to append the Homebrew initialization to your .zprofile:
>`echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile`
> (2) Activate the changes in your current shell session: `eval "$(/opt/homebrew/bin/brew shellenv)"`
> (3) Run the following command to check if Homebrew is recognized: `brew --version`
### Install Docker Desktop for Mac (Apple Silicon)
Download Docker Desktop for Mac with Apple Silicon support from [the official Docker website](https://www.docker.com/products/docker-desktop). This will also install the necessary virtualization support.
Ensure Docker Desktop is running after installation.

### Install kubectl:
Kubectl is a Kubernetes command-line tool that allows you to run commands against Kubernetes clusters. Install it using Homebrew:
```bash
brew install kubectl
```

### Install Minikube

Minikube is a tool that lets you run Kubernetes locally. Install it using Homebrew:
```bash
brew install minikube
```

## Set Up Minikube:

### Start Minikube:
Since you have Docker installed, you can use it as the driver for Minikube. This leverages native virtualization on M chip Macs:
```
minikube start --driver=docker
```

Note: This command starts a Minikube cluster using Docker. The first time you run this command, it will download necessary images, so it may take some time.

### Verify Installation:
After Minikube has successfully started, check the status of the cluster:
```
minikube status
```
Ensure that kubectl is configured to use the "minikube" context:
```
kubectl config use-context minikube
```

### Access Kubernetes Dashboard:
Minikube has a built-in dashboard. To access it:
```
minikube dashboard
```
This command will open up the Kubernetes dashboard in your default web browser.

### Enable Addons:

Minikube has several useful addons which you can enable or disable. For example, to enable metrics-server:
```
minikube addons enable metrics-server
```

### Stop and Start Minikube:
To stop Minikube:
```
minikube stop
```

To start it again:
```
minikube start
```

Deleting Minikube:
If you wish to delete your Minikube cluster and start fresh:
```
minikube delete
```

## Using Minikube:
You can now use kubectl to interact with your cluster. Start deploying applications, services, etc., using the kubectl command-line tool.

### Deploy Example Application:
Here's how you could create a simple deployment:
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
kubectl expose deployment hello-minikube --type=NodePort --port=8080
kubectl get services
```

Notes:

- Ensure you save this content into a file named README.txt.
- Adjust any paths or specific commands depending on your local setup or changes in software versions.

This document provides all the primary instructions needed to get Minikube running on an Apple Silicon Mac.
