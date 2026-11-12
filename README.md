# Kubernetes Pods – Hands-on Practical

## Objective

In this practical, you will learn how to:

1. Create and manage a Kubernetes Pod.
2. Inspect Pod details and status.
3. Access a container using `kubectl exec`.
4. View container logs.
5. Access an application using port forwarding.
6. Create a multi-container Pod.
7. Understand shared storage and networking between containers.
8. Understand Init Containers.
9. Observe how a Deployment replaces deleted Pods.

---

## Prerequisites

Make sure you have:

- A running Kubernetes cluster (Minikube, Kind, Docker Desktop Kubernetes, etc.).
- `kubectl` installed and configured.
- Basic knowledge of Docker containers and images.

Verify your setup:

```bash
kubectl version --client
kubectl cluster-info
kubectl get nodes
```

---

## Exercise 1: Create Your First Pod

We will create an Nginx Pod using a YAML manifest.

### Step 1: Create the YAML file

Create a file named `nginx-pod.yaml`.



### Step 2: Create the Pod

```bash
kubectl apply -f nginx-pod.yaml
```

### Step 3: Verify the Pod

```bash
kubectl get pods
kubectl get pods -o wide
```

### Step 4: Inspect the Pod

```bash
kubectl describe pod nginx-pod
```

Observe:
- Pod name
- Pod IP address
- Worker Node
- Container image
- Container status
- Events

---

## Exercise 2: Access the Container

### Step 1: View container logs

```bash
kubectl logs nginx-pod
```

### Step 2: Enter the container

```bash
kubectl exec -it nginx-pod -- sh
```

Execute the following commands inside the container:

```bash
whoami
hostname
pwd
ls
cat /etc/os-release
wget -qO- http://localhost
```

Exit the container:

```bash
exit
```

### Step 3: Access Nginx from your browser

Run this command in a separate terminal:

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

Open the following URL:

http://localhost:8080

You should see the Nginx welcome page.

Press `Ctrl + C` to stop port forwarding.

---

## Exercise 3: Create a Multi-Container Pod

### Objective

Create a Pod containing two containers:

- **Nginx:** Serves a web page.
- **BusyBox:** Updates the web page content every five seconds.

Both containers will share an `emptyDir` volume.

### Step 1: Create the YAML file

Create `multi-container-pod.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-web-pod
spec:
  volumes:
    - name: shared-data
      emptyDir: {}

  containers:
    - name: web
      image: nginx:alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html

    - name: content-writer
      image: busybox:1.36
      command: ["/bin/sh", "-c"]
      args:
        - |
          while true; do
            date > /data/index.html
            sleep 5
          done
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

### Step 2: Create the Pod

```bash
kubectl apply -f multi-container-pod.yaml
```

### Step 3: Check Pod status

```bash
kubectl get pods
kubectl describe pod shared-web-pod
```

Expected result:

```text
NAME             READY   STATUS
shared-web-pod   2/2     Running
```

### Step 4: View individual container logs

```bash
kubectl logs shared-web-pod -c web
kubectl logs shared-web-pod -c content-writer
