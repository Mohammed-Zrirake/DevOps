#DevOps #Containerization #Kubernetes #CoreConcept #Pods #kubectl #Debugging #HandsOn #Tutorial

>  Troubleshooting skills are essential when working with Kubernetes. The three primary `kubectl` commands for interacting with and debugging a [[The Kubernetes Pod|Pod]] are:
> 1.  **`kubectl describe pod <pod-name>`:** To see the event history and configuration.
> 2.  **`kubectl logs <pod-name>`:** To view the application's standard output (logs).
> 3.  **`kubectl exec -it <pod-name> -- /bin/bash`:** To get an interactive shell *inside* the running container.

---

This guide follows a hands-on demo of interacting with a running Pod. It assumes a Pod named `my-first-pod` has been created.

## 📊 Viewing Pod Logs (`kubectl logs`)

The first step in debugging is often to check the application's logs.

### 1. Dumping the Current Logs
The `kubectl logs` command will dump the entire log history of a container within a Pod to your terminal.
```bash
# First, get the exact name of your pod
kubectl get pods
# Alias: kubectl get po

# Then, dump its logs
kubectl logs my-first-pod
```
**Example Output:**
For an Nginx server, you'll see access logs for any requests that have been made.
```
10.244.0.1 - - [date] "GET / HTTP/1.1" 200 ...
10.244.0.1 - - [date] "GET /favicon.ico HTTP/1.1" 404 ...
```

### 2. Streaming Logs in Real-Time
To watch logs as they happen (like `tail -f` in Linux), use the `-f` or `--follow` flag.
```bash
kubectl logs -f my-first-pod
```
Now, if you access the application in your browser, you will see new log entries appear in your terminal in real-time. This is invaluable for live debugging.

---

## 🐚 Getting a Shell Inside a Container (`kubectl exec`)

Sometimes, viewing logs isn't enough. You need to get "inside" the container to inspect its filesystem, check running processes, or verify configuration files. The `kubectl exec` command allows you to do this.

### 1. Opening an Interactive Shell
To get an interactive shell session inside the container, use the `-it` flags.
```bash
kubectl exec -it my-first-pod -- /bin/bash
```
-   `exec`: The command to execute a command in a container.
-   `-i` (`--stdin`): Keep STDIN open, allowing you to type commands.
-   `-t` (`--tty`): Allocate a pseudo-TTY, which gives you a proper terminal interface.
-   `my-first-pod`: The name of the Pod you want to connect to.
-   `--`: This separator is important. It distinguishes the `kubectl` arguments from the command you want to run *inside* the container.
-   `/bin/bash`: The command to run. This starts a Bash shell. (If the container doesn't have `bash`, you might need to use `/bin/sh`).

**Result:**
Your terminal prompt will change, indicating you are now inside the container.
```
root@my-first-pod:/#
```
Now you can run standard Linux commands to explore the container's environment.
```bash
# List files in the current directory
ls -l

# Navigate to the web server's root directory
cd /usr/share/nginx/html

# View the content of the main page
cat index.html
```
This allows you to see the exact files that your web server is serving, confirming the content that is being displayed in the browser. Type `exit` to leave the container shell and return to your local terminal.

### 2. Executing a Single Command Remotely
If you don't need an interactive shell and just want to run a single command and see its output, you can specify it directly.

> [!warning] Deprecation Note
> The old syntax (`kubectl exec <pod> <command>`) is deprecated. The modern, correct syntax requires the `--` separator before the command.

**Modern Syntax:**
```bash
# Run the 'env' command inside the pod
kubectl exec my-first-pod -- env

# Run the 'ls -l' command
kubectl exec my-first-pod -- ls -l

# Cat a specific file inside the container
kubectl exec my-first-pod -- cat /usr/share/nginx/html/index.html
```
This is useful for quick checks and scripting.

---

## 🔬 Inspecting Pod State and Configuration

Beyond logs and shell access, you can get a wealth of information about a Pod's configuration and state.

### 1. `kubectl describe`
This is one of the most powerful troubleshooting commands. It provides a detailed, human-readable summary of a Kubernetes object, including its configuration, status, and most importantly, its recent **events**.
```bash
kubectl describe pod my-first-pod
```
The `Events` section at the bottom is the first place you should look when a Pod is failing to start.

### 2. Exporting the YAML Manifest
To see the complete, live configuration of a running Pod as a YAML manifest, use the `-o yaml` output flag.
```bash
kubectl get pod my-first-pod -o yaml
```
-   `-o` (`--output`): Specifies the output format. Other options include `json` and `wide`.
-   `wide`: The `kubectl get pods -o wide` command is useful for seeing additional information, such as the node the pod is scheduled on and its internal IP address.

---

## 🧹 Cleaning Up Resources

It is a critical best practice to delete resources after you are finished with them to avoid unnecessary costs and to keep your cluster clean.

### 1. Get a List of All Resources
The `kubectl get all` command shows you all the core resources (Pods, Services, Deployments, ReplicaSets) in the current namespace (which is `default` unless specified otherwise).
```bash
kubectl get all
```

### 2. Deleting Resources
You can delete resources by their type and name.
```bash
# Delete the Pod
kubectl delete pod my-first-pod

# Delete the Service
kubectl delete service my-first-service
```
You can also specify the type and name together:
```bash
kubectl delete service/my-first-service
```

### 3. Verifying Deletion
Run `kubectl get all` again. You should see that your pod and service are gone (only the default `service/kubernetes` should remain).

> [!info] When you delete a `LoadBalancer` service, the Kubernetes controller for your cloud provider (Azure in this case) will automatically de-provision the associated cloud resources, such as the Public IP address and the Load Balancing Rule, cleaning everything up for you.