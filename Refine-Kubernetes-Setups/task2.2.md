# Secrets

In Kubernetes, **Secrets** are objects that let you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. Using Kubernetes Secrets is preferable to storing sensitive data in your Pod definitions or in container images, as it reduces the risks associated with data exposure and offers more control.

### Example Scenario: Using Secrets

Let's say you have an application that needs access to a database password, and you want to manage this securely within Kubernetes.

#### Step 1: Creating the Secret

You need to first create a Secret that contains the required sensitive data. You can do this either via a YAML file or through the `kubectl` command line.

**Using `kubectl` from the command line:**

To create a Secret directly through kubectl which stores a database password:

```bash
kubectl create secret generic db-secret --from-literal=password=mysecretpassword
```

This command creates a Secret named `db-secret` with the data `"password=mysecretpassword"`.

**Using a YAML file:**

First, encode your secret using base64 encoding to ensure that the Secret contents are not easily readable:

```bash
echo -n 'mysecretpassword' | base64
```
Output will be something like: `bXlzZWNyZXRwYXNzd29yZA==`

Next, create a YAML file named `db-secret.yaml` with the following content:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlzZWNyZXRwYXNzd29yZA==
```

Apply it with `kubectl`:

```bash
kubectl apply -f db-secret.yaml
```

#### Step 2: Using the Secret in a Pod

Now, let's use this secret in a Pod. You can inject the secret into your pods using environment variables or as files in a volume.

**As an environment variable:**

Create a Pod that uses the secret as an environment variable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    args:
      - sleep
      - "3600"  # Keeps the pod alive for 3600 seconds (1 hour)
    env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-secret
            key: password
  restartPolicy: Never
```

```bash
kubectl apply -f secret-demo-pod.yaml
```

This configuration mounts the Secret `db-secret` and exposes the `password` data as an environment variable `DB_PASSWORD`.

**As a file in a volume:**

If you prefer to mount Secrets as files:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command:
      - sleep
      - "3600"  # Keeps the pod alive for 3600 seconds (1 hour)
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
  restartPolicy: Never
```

```bash
kubectl apply -f secret-volume-pod.yaml
```

This configuration will mount the secret data as files under the `/etc/secret` directory. Each key in the Secret becomes a separate file with the filename corresponding to each key and the content of the file being the data associated with that key. In this case, the `password` key in the `db-secret` secret will appear as a file in `/etc/secret/password`.

### Step 3: Confirming the Secret is Mounted Correctly

You can verify if the secret is accessible to the pod in the desired format (either environment variable or file):

1. **Check environment variable:**
   Use `kubectl exec` to execute a command inside the pod that reads the environment variable:

   ```bash
   kubectl exec secret-demo-pod -- env | grep DB_PASSWORD
   ```

   This command will output the environment variable `DB_PASSWORD` that you have set in the pod, and you should see the password value `mysecretpassword`.

2. **Check mounted file:**
   Similarly, you can check if the secret was correctly mounted as a file:

   ```bash
   kubectl exec secret-volume-pod -- cat /etc/secret/password
   ```

   This command will read the content of the file where the secret is mounted, displaying the decoded password, which should be `mysecretpassword`.

### Step 4: Managing Secrets

Managing Kubernetes Secrets involves more than just creating and using them. Best practices to consider include:

- **RBAC (Role-Based Access Control):** Make sure access to Secrets is tightly controlled by using Kubernetes RBAC. This involves setting permissions that restrict which users and components of your system can access the Secrets.

- **Encryption at Rest:** Enable encryption at rest to protect your secrets from being read if the underlying storage mechanism is compromised. 

- **Automated Rotation:** Implement automated processes to rotate secrets periodically and thus reduce the risk of exposure from long-standing secrets. Ensure your application can handle secret rotation seamlessly.

- **Limit secret usage:** Be considered about who or what is using your secrets. Applying the principle of least privilege, make sure that only necessary entities have access to the secrets they need.
