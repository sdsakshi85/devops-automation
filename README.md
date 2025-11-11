### Link for K8S Setup (worker and master)

https://github.com/LondheShubham153/kubestarter/tree/main/Kubeadm_Installation_Scripts_and_Documentation

### Security Group Inbound Rules
![This is my project screenshot](./images/securitygroup.jpg)

### Step 1: Create the Namespace

First, create the new namespace on your cluster:

```bash
kubectl create namespace database-stack
```

-----

### Step 2: Prepare the Worker Node for Storage

This step is the same as before. You still need to create the `hostPath` directory on the worker node that will physically store the data.

1.  SSH into your worker node.
2.  Create the directory and set permissions.

<!-- end list -->

```bash
# On your WORKER node
ssh user@your-worker-node-ip
sudo mkdir -p /mnt/data/postgres
sudo chmod 777 /mnt/data/postgres
```

-----

### Step 3: Create the Kubernetes Manifest File

Save the following as `postgres-stack.yaml`. This file is identical to the previous one; it contains all seven resources. We will tell `kubectl` which namespace to put them in when we run the `apply` command.


### Step 4: Apply the Manifest (in the new namespace)

This is the key difference. Use the `-n` (or `--namespace`) flag to apply this file.

```bash
kubectl apply -f postgres-stack.yaml -n database-stack
```

The `-n database-stack` flag tells `kubectl` to create all the namespaced resources (everything *except* the `PersistentVolume`) inside your new namespace.

-----

### Step 5: Verify Everything is Running

Now, you must add the `-n database-stack` flag to your `kubectl get` commands to see the resources.

1.  **Check the PV (no namespace needed) and PVC (namespace needed):**

    ```bash
    # PV is cluster-wide, no -n flag needed
    kubectl get pv

    # PVC is namespaced
    kubectl get pvc -n database-stack
    ```

    The `STATUS` for both should be **Bound**.

2.  **Check the pods, services, and deployments:**

    ```bash
    # You can get them all at once
    kubectl get pods,svc,deploy,secret -n database-stack
    ```

    Wait for both `postgres-deployment` and `adminer-deployment` pods to be **Running**. Note the `NodePort` for the `adminer-service`.

-----

### Step 6: Access Adminer and Connect

This part is exactly the same as before.

1.  Find your Adminer `NodePort` from the `kubectl get svc -n database-stack` command.
2.  Open your browser to `http://<your-worker-node-ip>:<node-port>`.
3.  Use the following login details:
      * **System:** `PostgreSQL`
      * **Server:** `ClusterIP` (ClusterIP of postgres-service of type ClusterIP)
      * **Username:** `postgres`
      * **Password:** `mysecretpassword`
      * **Database:** `postgres`

